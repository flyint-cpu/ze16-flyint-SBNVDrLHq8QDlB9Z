
11月26日在华为Mate品牌盛典上，全新Mate70系列及多款全场景新品正式亮相。在游戏领域，HarmonyOS NEXT加持下游戏的性能得到充分释放。HarmonyOS SDK为开发者提供了软硬协同的系统级图形加速解决方案\-\-\-\-\-\-Graphics Accelerate Kit（图形加速服务），帮助游戏应用快速集成超帧、GTX自适应稳态渲染和自适应刷新率等渲染优化能力，并解决游戏运行不流畅、卡顿掉帧等痛点体验问题，让接入HarmonyOS SDK的手游，如《决胜巅峰》等，在Mate新机上的体验实现大幅提升。


![image](https://img2024.cnblogs.com/blog/2396482/202412/2396482-20241202154253113-824442292.png)


**独特大视野，推塔体验细腻又流畅**


《决胜巅峰》在HarmonyOS NEXT上首发，并且在Mate X6折叠屏上首次以大视野和高帧率的形式呈现给MOBA类游戏玩家。在Mate X6折叠屏手机的超大视野下，《决胜巅峰》的可视画面提升25%，能够更全面地观察战场。通过超帧技术，将帧率突破到120帧，使得每一次技能释放、每一次推塔，都显得更加细腻、顺畅，极大地提升了游戏的沉浸感，为玩家带来前所未有的对战快感。


在对快速反应要求极高的MOBA类游戏中，超帧技术带来的120帧效果让画面更为流畅，帧率也更稳定，帮助玩家更好地掌控局势，提升整体的游戏体验。在团战中，华丽的技能和流畅的动画结合，也使得战斗画面更加震撼。


**优化性能开销，游戏效果不 "打折"**


通过结合《决胜巅峰》游戏开发者主动提供的游戏过程关键信息，可以实现GTX自适应稳态渲染和自适应刷新率等游戏加速方案，帮助开发者打造高画质、高流畅、低功耗的非凡游戏体验。


**自适应稳态渲染**可根据游戏场景、GPU负载等信息，动态调整渲染负载，减少游戏掉帧，让玩家获得一个更加稳定的游戏体验。


![image](https://img2024.cnblogs.com/blog/2396482/202412/2396482-20241202154321507-1946643827.png)


自适应稳态渲染技术接入架构示意图


**自适应稳态渲染接入代码：**



```
// 初始化ABR实例，配置Buffer分辨率因子范围属性，结合具体游戏分辨率、画质设置合适的范围
// 例如设置ABR对Buffer分辨率进行0.8~1.0倍的自适应调整
errorCode = HMS_ABR_SetScaleRange(context_, 0.8f, 1.0f);
if (errorCode != ABR_SUCCESS) {
    GOLOGE("HMS_ABR_SetScaleRange execution failed, error code: %d.", errorCode);
    return false;
}
// 激活ABR上下文实例
errorCode = HMS_ABR_Activate(context_);
if (errorCode != ABR_SUCCESS) {
    GOLOGE("HMS_ABR_Activate execution failed, error code: %d.", errorCode);
    return false;
}
// 相机运动数据结构体，设置每帧实时相机运动数据
ABR_CameraData cameraData;
cameraData.position = static_cast(camera_.GetPosition());
cameraData.rotation = static_cast(camera_.GetRotation());
// 每帧相机运动数据更新
errorCode = HMS_ABR_UpdateCameraData(context_, &cameraData);
if (errorCode != ABR_SUCCESS) {
    GOLOGE("HMS_ABR_UpdateCameraData execution failed, error code: %d.", errorCode);
    return false;
}
//渲染前的准备，绑定目标帧缓冲索引，清空颜色缓冲 renderer_->BeginRenderTarget(fbo,BACKGROUND.x_, BACKGROUND.y_, BACKGROUND.z_, 1.0F); 
//在Buffer渲染前调用，执行失败不影响Buffer正常渲染 
errorCode = HMS_ABR_MarkFrameBuffer_GLES(context_);
if (errorCode != ABR_SUCCESS) {
    GOLOGE("HMS_ABR_MarkFrameBuffer_GLES execution failed, error code: %d.", errorCode);
}
//调用绘制方法进行渲染 
opaqueLayer_Render(sceneDelta, camera_.GetviewMatrix() .lastViewProj_); 
//获取每帧的缩放信息 
float scale; 
erorCode =HMS_ABR_GetScale(context_. &scale); 
GOLOGD("Scale is %f, ". scale); 
if (errorCode != ABR_SUCCESS) {
GOLOGE("HMS_ABR_GetScale execution failed, error code: %d.", errorCode);
}
//将帧缓冲索引绑定为默认值0 
renderer_->EndRenderTarget();

```

![image](https://img2024.cnblogs.com/blog/2396482/202412/2396482-20241202154340299-1123991573.jpg)


[点击查看自适应稳态渲染接入教程](https://github.com)


据统计，多数游戏120FPS档位下，其实约70%渲染场景是没有变化的，因此主场景里将近7成的渲染都并非是必要的。


**自适应刷新率**可以根据游戏场景、玩家操作、CPU负载、GPU负载、手机温度等信息，智能地调整游戏帧率与屏幕刷新率，保证体验基础上实现降低功耗。在玩家休闲状态、游戏画面变化幅度小的情况下，帧率可适当调低，节省电量、控制温度；而在玩家操作剧烈、游戏画面急速变化时，帧率则提升到极致水平，保证玩家的游戏体验。


![image](https://img2024.cnblogs.com/blog/2396482/202412/2396482-20241202154414878-552468198.png)


自适应刷新率接入架构示意图


![image](https://img2024.cnblogs.com/blog/2396482/202412/2396482-20241202154424891-1142767731.png)


自适应刷新率接入流程示意图


**自适应刷新率接入代码：**



```
//配置OpenGTX属性
errorCode=HMS_OpenGTX_SetConfiguration(contextGtx_, &configGtx);
if (errorCode != OPENGTX_SUCCESS) {
   GOLOGE("HMS_OPENGTX_SetConfiguration execution failed, error code: %d.", errorCode);
   return false;
}
//激活OpenGTX上下文实例
errorCode = HMS_OpenGTX_Activate(contextGtx_);
if (errorCode != OPENGTX_SUCCESS) {
   GOLOGE("HMS_OpenGTX_Activate execution failed, error code: %d.", errorCode);
   return false;
}
//发送游戏场景信息
OpenGTX_GameScenelnfo gameScenelnfo;
gameScenelnfo.scenelD=OTHERS_SCENE;
gameScenelnfo.description=OGBT_DESCRIPTION.data();
gameScenelnfo.recommendFPS=OGBT_RECOMMEND_FPS;
gameScenelnfo.minFPS=OGBT_MIN_FPS;
gameScenelnfo.maxFPS=OGBT_MAX_FPS;
gameScenelnfo.resolutionCurValue.height=OGBT_RES_HEIGHT;
gameScenelnfo.resolutionCurValue.width=OGBT_RES_WIDTH;
errorCode = HMS_OpenGTX_DispatchGameSceneInfo(contextGtx_, &gameSceneInfo);
if (errorCode != OPENGTX_SUCCESS) {
   GOLOGE("HMS_OpenGTX_DispatchGameSceneInfo execution failed, error code: %d.", errorCode);
   return false;
}
//发送游戏网络信息
OpenGTX_Networklnfo networklnfo;
networkInfo.networkLatency.down = OGBT_NETWORK_LATENCY_DOWN;
networklnfo.networkLatency.up= OGBT_NETWORK_LATENCY_UP;
networklnfo.networkLatency.total =OGBT_NETWORK_LATENCY_TOTAL;
networkInfo.networkServerlP=OGBT_NETWORK_SERVER_IP.data();
errorCode = HMS_OpenGTX_DispatchNetworklnfo(contextGtx_, &networklnfo);
if (errorCode != OPENGTX_SUCCESS) {
   GOLOGE("HMS_OpenGTX_ DispatchNetworklnfo execution failed, error code: %d.", errorCode);
   return false;
}

```

![image](https://img2024.cnblogs.com/blog/2396482/202412/2396482-20241202154442756-1786499759.jpg)


[点击查看自适应刷新率接入教程](https://github.com):[wgetcloud全球加速器服务](https://wgetcloud6.org)


除了图形加速服务（Graphics Accelerate Kit），HarmonyOS SDK还提供更多图形开放能力，为游戏等重载应用的图形渲染提供帮助，帮助开发者打造高帧率、高画质、低功耗的用户体验。


**探索更多**


访问[图形加速服务](https://github.com)（Graphics Accelerate Kit），了解更多详情开始使用。


\**本文所提及数据均为内部实验室测试结果*


\**本文引用素材来自三方授权，因版本/机型不同可能存在变化，以实际为准。*


