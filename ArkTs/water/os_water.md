# 鸿蒙水印组件


```javascript
import { display, matrix4 } from '@kit.ArkUI';
import measure from '@ohos.measure'
import { emitter } from '@kit.BasicServicesKit';

///明水印标示 color:@"明水印B" alpha:0.16 space:130
// const  ObviousWaterMarkImageViewTag:number = 20180523;
///暗水印标示 color:@"0x暗水印" 0.01 space:90
// const  OffColorWaterMarkImageViewTag:number = 20180524;
// 水印格式 "名字 电话后四位 日期 网络环境"，如 eg：水印 0202 202400428 网络环境
// let storage = LocalStorage.getShared()
@Entry
@Component
export struct WaterMark {
  @StorageProp('waterMarkContent') @Watch('getPositions') content:string = "";
  // @State  content:string = "";
  @Prop fontSize:number = 14;
  @Prop color:string = '#BDBDBD';
  @Prop alpha:number = 0.01;
  @Prop space:number= 90;
  @State positions:Array<Position> = [];
  aboutToAppear(): void {
    //Test TODO
    // let innerEvent: emitter.InnerEvent = { eventId: 3533 }
    // emitter.on(innerEvent, data => {
    //   Logger.info(' emitter.InnerEvent 3533 data:',JSON.stringify(data));
    //   if(data.data instanceof Object){
    //     this.content = data.data['waterMarkContent'];
    //     this.getPositions();
    //   }
    // })
    this.getPositions();
  }
  build() {
    Column() {
      ForEach(this.positions,(item:Position)=>{
        Text(this.content)
          .position(item)
          .fontSize(this.fontSize)
          .fontColor(this.color)
          .transform(matrix4.identity().rotate({z:1,angle: this.alpha<=0.02?90:-45 })) //(this.alpha<=0.02?Math.PI/2:-Math.PI/4)*360
      })
    }.padding({left:40,right:40,top:40,bottom:80})
    .width('100%')
    .height('100%')
    .backgroundColor(Color.Transparent)
    .opacity(this.alpha)
    .enabled(false)
  }
  getPositions(){
    if(this.content.length>0){
      let displayConfig: display.Display = display.getDefaultDisplaySync();
      let waterMarkPositions:Array<Position> = [];
      let  textSize:SizeOptions= measure.measureTextSize({
        textContent: this.content,
        fontSize: this.fontSize
      })
      let textWidth:number = px2vp(parseInt(textSize.width as string)) ;
      let textHight:number = px2vp(parseInt(textSize.height as string)) ;
      let positionY = - 30;
      while (positionY<= displayConfig.width){
        let positionX = -30;
        while (positionX<= displayConfig.height){
          waterMarkPositions.push({x:positionX,y:positionY});
          positionX = positionX + textWidth/Math.SQRT2 + this.space;
        }
        positionY = positionY + textHight/Math.SQRT2 + this.space;
      }
      this.positions = waterMarkPositions;
    }
  }
}
```
使用示例：
可以放在整个app的根上，也可以加到单独页面上

```javascript
Stack(){
	OtherWidget（其它Widget组件）
	WaterMark({color:"#BDBDBD",space:100,alpha:0.2}) //水印，可以多个叠加
	WaterMark({color:"#B6B6BB",space:70})
}
```
