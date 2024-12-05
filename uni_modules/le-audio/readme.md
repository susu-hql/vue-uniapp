# le-audio

### 使用方式

```javascript
<le-audio
    :activeIndex="audioActiveIndex"
    :audioData="audioList"
    :showAudioListIcon="showAudioListIcon"
	:showAudioSpeedIcon="showAudioSpeedIcon"
	:autoplay="autoplay"
    @onOpenAudioList="onOpenAudioList"
    @onAudioChange="onAudioChange"
></le-audio>
```

### 组件的属性说明如下：

| 属性               | 类型    | 默认值 | 必填 | 说明                                                   |
| ------------------ | ------- | ------ | ---- | ------------------------------------------------------ |
| audioData          | Array   | []     | 是   | 播放列表数据                                           |
| activeIndex        | Number  | -1     | 否   | 默认播放的索引                                         |
| showAudioListIcon  | Boolean | false  | 否   | 是否显示播放列表按钮（仅是一个按钮，不会弹出真的列表） |
| showAudioSpeedIcon | Boolean | false  | 否   | 是否显示播放倍速按钮                                   |
| autoplay           | Boolean | true   | 否   | 初始化时是否自动播放                                   |

#### audioData参数说明：

| 属性    | 类型   | 必填 | 说明      |
| ------- | ------ | ---- | --------- |
| title   | String | 是   | 播放名称  |
| fileUrl | String | 是   | 播放链接  |
| image   | String | 否   | 封面图url |
| epname  | String | 否   | 专辑名    |
| singer  | String | 否   | 歌手名    |
| webUrl  | String | 否   | 页面链接  |

### 事件

| 事件名称       | 回调参数             | 说明                                                  |
| -------------- | -------------------- | ----------------------------------------------------- |
| onOpenAudioLis | () => void           | 点击播放列表按钮事件                                  |
| onAudioChange  | (data,index) => void | 改变事件，data为当前播放的数据，index为当前播放的索引 |



