<img src="https://raw.githubusercontent.com/xueliangwd/leon/main/images/blog_header.jpg" alt="" title="">

# 日视图效果算法实现

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [日视图效果算法实现](#日视图效果算法实现)
      - [效果分析：](#效果分析)
      - [定义一个树节点的Node，ScheduleNode：](#定义一个树节点的nodeschedulenode)
      - [日程时间是否有重叠](#日程时间是否有重叠)
      - [获取最深层的日程：](#获取最深层的日程)
      - [设置Schedule所占宽度权重：](#设置schedule所占宽度权重)
      - [核心排练日程的位置计算规则](#核心排练日程的位置计算规则)
      - [整体排序：](#整体排序)

<!-- /code_chunk_output -->


<img src="https://raw.githubusercontent.com/xueliangwd/leon/main/images/schedule_day.jpg" alt="" title="">

实现如上图中，类似钉钉、飞书中日历日视图效果；
#### 效果分析：
在y轴上高度代表时间间隔，在x轴上有时间交叉的日程进行宽度等分；
一个日程能与多个日程产生时间交叉，所以我们不能用线性结构处理，这样我们就考虑用树或图结构；
一个日程交叉点会大于2个，这里采用多叉树（如果不太好理解，枚举较全面的效果，顺时针转90度看效果图也很像一个树结构）；

#### 定义一个树节点的Node，ScheduleNode：
  ```dart
  ScheduleNode? parent;
    List<ScheduleNode>? children;
    double x = 0; // UI中x轴位置 positionX = x*WidgetWidth
    int? beginTime; // 时间戳 开始时间在UI中计算y位置
    int startMinute; // 相对00：00开始的分钟
    int duration;  // 分钟
    double weight = 1;//UI中宽度权重 width=weight*WidgetWidth
    String? title;
    // 同一时间的任务用link连接起来
    int level = 1;//层级,用于计算作为父节点时应该平分多少宽度
```

#### 日程时间是否有重叠
```dart
 bool isTaskOverlap(ScheduleNode taskB) {
    ScheduleNode taskA = this;
        if (taskA == taskB) return false;
        int taskAStart = taskA.getStartMinute();
        int taskAEnd = taskAStart + taskA.getDuration();
        int taskBStart = taskB.getStartMinute();
        int taskBEnd = taskBStart + taskB.getDuration();
        if (taskBStart > taskAStart && taskBStart < taskAEnd ||
            taskBEnd > taskAStart && taskBStart < taskAEnd ||
            taskBStart == taskAStart && taskBStart == taskAEnd ) {
          return true;
        } else {
          return false;
        }
   }
```
#### 获取最深层的日程：
```dart
ScheduleNode getRearItem(ScheduleNode maxItem) {
	ScheduleNode p = this;
	ScheduleNode q = maxItem;
	if(p.children != null&& (p.children?.length??0)>0){
		for(ScheduleNode child in p.children!){
			q = child.getRearItem(q);
		}
	}else{
		q = p;
	}
	return q.level > maxItem.level?q:maxItem;
}
```
#### 设置Schedule所占宽度权重：
```dart
    void setLinkWeight() {
    	ScheduleNode p = this;
    	ScheduleNode maxTask = getRearItem(p);
    	if(p.parent == null){
    			p.weight = 1.0/maxTask.level;
    	}else{
    			p.x = p.parent!.x + p.parent!.weight;
    			p.weight = (1.0 - p.x )/(maxTask.level - p.level + 1);
    	}
    	if(p.children != null){
    		for(ScheduleNode child in p.children!){
    			child.setLinkWeight();
    		}
    	}
}
```
#### 核心排练日程的位置计算规则
```dart
List<ScheduleNode> _arrangeTaskDataNew(List<ScheduleNode> tasks) {
	//按日程时间排序
	tasks.sort((ScheduleNode left, ScheduleNode right) {
	if (left.getDuration() == right.getDuration()) {
		return  left.getStartMinute().compareTo(right.getStartMinute());
	} else {
		return  - left.getDuration().compareTo(right.getDuration());
	}
	});
	//遍历将有时间重叠的日程关联分组
	for (int i = 0; i < tasks.length; i++) {
		ScheduleNode taskA = tasks[i];
		for (int j = i + 1; j < tasks.length; j++) {
			if (tasks[i].isTaskOverlap(tasks[j])) {
				ScheduleNode taskB = tasks[j];
				if(taskB.parent ==null){
					taskB.level = taskA.level +1;
					taskB.parent = taskA;
					taskA.children == null?taskA.children = [taskB]:taskA.children!.add(taskB);
				}else if(taskB.level < taskA.level +1){
					taskB.parent!.children!.remove(taskB);
					taskB.level = taskA.level +1;
					taskB.parent = taskA;
					taskA.children == null?taskA.children = [taskB]:taskA.children!.add(taskB);
				}
			}
		}
	}
//同行日程数过多 拆分  完全相同时间优化
// for (int i = 0; i < tasks.length; i++) {
//   ScheduleNode taskA = tasks[i];
//   if(taskA.level > 7){
//     //寻找拆分的父节点
//     ScheduleNode taskB = taskA.getSplitItem();
//     if(taskB != null){
//       //重设该节点层级
//       ScheduleNode parent = taskB.getInsertParentItem(taskB);
//       taskB.parent.children.remove(taskB);
//       taskB.parent = parent;
//       taskB.level = parent != null?parent.level+1:1;
//       taskB.setLinkLevel();
//     }
//   }
// }
	//标记日程链宽度位置
	for (int i = 0; i < tasks.length; i++) {
		ScheduleNode taskA = tasks[i];
		if(taskA.parent == null){
			// // 获取最长层级
			// ScheduleNode maxTask = taskA.getRearItem(taskA);
			//根据最长层级，计算宽度和位置
			taskA.setLinkWeight();
		}
	}
	return tasks;
}
```
#### 整体排序：
```dart
List<ScheduleNode> arrangeTaskData(List<ScheduleNode> taskList) {
	List<ScheduleNode> tasks = _arrangeTaskDataNew(taskList);
	for (final task in tasks) {
		if (task.x + task.weight > 1) {
			if (task.x < 1) {
				task.weight = 1  - task.x;
			} else {
				task.x = 0;
				task.weight = 1;
			}
		}
	}
	return tasks;
}
```