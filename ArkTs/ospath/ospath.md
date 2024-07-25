# 鸿蒙图片保存读取


```javascript
import picker from '@ohos.file.picker';
import { photoAccessHelper } from '@kit.MediaLibraryKit';
import fs from '@ohos.file.fs';
import { common, Permissions } from '@kit.AbilityKit';
import { image } from '@kit.ImageKit';
import { http } from '@kit.NetworkKit';
import ResponseCode from '@ohos.net.http';
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import { Size } from '@kit.ArkUI';


interface PhotoDic{
  typeString?:number;
  localUrl:string;
  isOriginal:boolean;
  // @"length":dic[@"fileSize"],
  // @"width":dic[@"width"],
  // @"height":dic[@"height"],
  // @"duration":dic[@"videoDuraiton"],
  // @"thumbpath":thumbPath,
  // @"mediaType":@"mp4",
}
const PERMISSIONS: Array<Permissions> = [
  'ohos.permission.READ_MEDIA',
  'ohos.permission.WRITE_MEDIA',
  'ohos.permission.WRITE_IMAGEVIDEO'
];

export class PhotoManager {
  // image: PixelMap | undefined = undefined;
  // imageBuffer: ArrayBuffer | undefined = undefined; // 图片ArrayBuffer

  // 将公共路径文件复制到沙箱
  async copyPublicToSandbox(uri:string):Promise<string> {
    try {
      const context = getContext(this) as common.UIAbilityContext;
      let sandBoxDir = context.tempDir;
      let uriArr:string[] = uri.split("/");
      let name:string = '';
      if(uriArr.length>2){
        name = uriArr[uriArr.length-1];
      }
      let dest = sandBoxDir +"/box"+name;
      // fs.copyFileSync(uri,dest);
      let file = fs.openSync(uri, fs.OpenMode.READ_WRITE);
      fs.copyFileSync(file.fd,dest);
      fs.closeSync(file);
      return dest;
    } catch (err) {
      console.info('[demo] copyPublic2Sandbox fail, err: ', err);
      return Promise.reject(err);
    }
  }
  /**
   * 保存ArrayBuffer到图库
   * @param buffer：图片ArrayBuffer
   * @returns
   */
   async saveImage(buffer: ArrayBuffer | string): Promise<void> {
    try{
      const context = getContext(this) as common.UIAbilityContext;
      const atManager = abilityAccessCtrl.createAtManager();
      await atManager.requestPermissionsFromUser(context, PERMISSIONS);//
      const helper = photoAccessHelper.getPhotoAccessHelper(context); // 获取相册管理模块的实例
      const uri = await helper.createAsset(photoAccessHelper.PhotoType.IMAGE, 'jpg'); //需获取ohos.permission.WRITE_IMAGEVIDEO权限 指定待创建的文件类型、后缀和创建选项，创建图片或视频资源
      const file = await fs.open(uri, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
      await fs.write(file.fd, buffer);
      await fs.close(file.fd);
    }catch(e){
      console.info('copySandboxToPublic fail, err: ', e);
    }
  }
  // 将沙箱路径文件复制到相册
  async copySandboxToPublic(imgPath:string) {
    try{
      let file = fs.openSync(imgPath, fs.OpenMode.READ_WRITE);
      //获取文件buffer
      let size = fs.statSync(file.fd).size;
      let imgBuffer = new ArrayBuffer(size);//fs.statSync(file.fd).size
      let resultSize = fs.readSync(file.fd,imgBuffer);
      console.info('copySandboxToPublic size: ', size,' -resultSize:',resultSize);
      fs.close(file);
      await this.saveImage(imgBuffer);
    } catch (e) {
      console.info('copySandboxToPublic fail, err: ', e);
    }
  }
  /**
   * 通过http的request方法从网络下载图片资源
   */
  // async getPicture() {
  //   http.createHttp()
  //     .request('https://gitee.com/harmonyos-cases/cases/raw/master/CommonAppDevelopment/feature/variablewatch/src/main/resources/base/media/grape.png',
  //       (error: BusinessError, data: http.HttpResponse) => {
  //         if (error) {
  //           // 下载失败时弹窗提示检查网络，不执行后续逻辑
  //           promptAction.showToast({
  //             message: '图片加载失败，请检查网络',
  //             duration: 2000
  //           })
  //           return;
  //         }
  //         this.transcodePixelMap(data);
  //         // 判断网络获取到的资源是否为ArrayBuffer类型
  //         if (data.result instanceof ArrayBuffer) {
  //           this.imageBuffer = data.result as ArrayBuffer;
  //         }
  //       }
  //     )
  // }

  /**
   * 使用createPixelMap将ArrayBuffer类型的图片装换为PixelMap类型
   * @param data：网络获取到的资源
   */
  static async transcodePixelMap(data: http.HttpResponse):Promise<PixelMap> {
    if (ResponseCode.ResponseCode.OK === data.responseCode) {
      const imageData: ArrayBuffer = data.result as ArrayBuffer;
      // 通过ArrayBuffer创建图片源实例。
      const imageSource: image.ImageSource = image.createImageSource(imageData);
      const options: image.InitializationOptions = {
        'alphaType': 0, // 透明度
        'editable': false, // 是否可编辑
        'pixelFormat': 3, // 像素格式
        'scaleMode': 1, // 缩略值
        'size': { height: 100, width: 100 }
      }; // 创建图片大小

      // 通过属性创建PixelMap
      let pixelMap =await imageSource.createPixelMap(options);
      return pixelMap;
    }
    return Promise.reject('error');
  }

  /**
   * *mimeType PhotoViewMIMETypes 可选择的媒体文件类型，若无此参数，则默认为图片和视频类型。
   * IMAGE_TYPE  图片类型
   * VIDEO_TYPE 视频类型
   * IMAGE_VIDEO_TYPE 图片和视频类型
   *
   * *maxSelectNumber number 选择媒体文件数量的最大值(默认值为50，最大值为500)。
   *
   * *return PhotoSelectResult 图库选择后的结果集
   * photoUris Array<string>  返回图库选择后的媒体文件的uri数组
   * isOriginalPhoto boolean  返回图库选择后的媒体文件是否为原图
   */
  static async photoViewPicker(mimeType?: picker.PhotoViewMIMETypes, maxSelectNumber?: number):Promise<picker.PhotoSelectResult> {
    try {
      let PhotoSelectOptions = new picker.PhotoSelectOptions();
      PhotoSelectOptions.MIMEType = mimeType;
      PhotoSelectOptions.maxSelectNumber = maxSelectNumber;
      let photoPicker = new picker.PhotoViewPicker();
      photoPicker.select(PhotoSelectOptions).then((PhotoSelectResult) => {
        console.info('PhotoViewPicker.select successfully, PhotoSelectResult uri: ' + JSON.stringify(PhotoSelectResult));
        return PhotoSelectResult;
      }).catch((err: string) => {
        console.error('PhotoViewPicker.select failed with err: ' + err);
        // return Promise.resolve(new picker.PhotoSelectResult);
        // return Promise.reject(err);
        return new picker.PhotoSelectResult;
      });
    } catch (err) {
      console.error('PhotoViewPicker failed with err: ' + err);
      // return Promise.resolve(new picker.PhotoSelectResult);
      // return Promise.reject(err);
      return new picker.PhotoSelectResult;
    }
    return new picker.PhotoSelectResult;
    // return Promise.reject(new picker.PhotoSelectResult);
  }

  static async photoDic(photoSelectResult:picker.PhotoSelectResult,mimeType:picker.PhotoViewMIMETypes): Promise<Array<PhotoDic>>{
    let result: Array<PhotoDic> = [];
    let phm =new PhotoManager();
    for(let uri  of photoSelectResult.photoUris){
      try{
        let imagePath =await phm.copyPublicToSandbox(uri);
        //获取图片 宽高 大小 TODO
        result.push({
          'typeString':mimeType==picker.PhotoViewMIMETypes.VIDEO_TYPE?1:0,//TODO
          'localUrl':imagePath,
          'isOriginal':photoSelectResult.isOriginalPhoto
        });
      }catch (e){
        console.error('photoDic failed with err: ' + e);
      }
    }
    return result;
  }
  //传入文件path或uri 返回文件大小
  getFileByteSize(path:string):number{
    let file = fs.openSync(path, fs.OpenMode.READ_ONLY);
    //获取文件buffer
    let size = fs.statSync(file.fd).size;
    fs.close(file.fd);
    return size;
  }
  //传入文件path或uri 返回图片size（width height）
  async getImageSize(path:string):Promise<Size>{
    let file = fs.openSync(path, fs.OpenMode.READ_ONLY);
    const imageSourceApi : image.ImageSource = image.createImageSource(file.fd);
    fs.close(file.fd);
    try{
      let imageInfo : image.ImageInfo =await imageSourceApi.getImageInfo(0);
      return imageInfo.size;
    }catch (e){
      console.error('Failed to obtain the image information.');
      return Promise.reject(e);
    }
  }
  /**
   *   返回documentPicker选择后的结果集
   */
  static async documentViewPicker(): Promise<Array<string> | void> {
    try {
      let DocumentSelectOptions = new picker.DocumentSelectOptions();
      let documentPicker = new picker.DocumentViewPicker();
      documentPicker.select(DocumentSelectOptions).then((DocumentSelectResult) => {
        console.info('DocumentViewPicker.select successfully, DocumentSelectResult uri: ' + JSON.stringify(DocumentSelectResult));
        return DocumentSelectResult;
      }).catch((err: string) => {
        console.error('DocumentViewPicker.select failed with err: ' + err);
        return Promise.reject(err);
      });
    } catch (err) {
      console.error('DocumentViewPicker failed with err: ' + err);
      return Promise.reject(err);
    }
  }

}
```

