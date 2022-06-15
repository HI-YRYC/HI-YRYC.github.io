## 中间件数据查询
1. redis
-- 命令行进入
```
redis-cli -h 127.0.0.1 -p 6379
```
-- DOCKER进入
```
docker exec -it id redis-cli
```
-- 若有密码
```
auth 密码
```
-- 查询
```
select 1
keys *
get key
```
2. mongo
-- 命令行进入
```
mongo
```
-- DOCKER进入
```
docker exec -it id mongo
```
-- 查询
```
show dbs
use database
show tables
db.tableName.find({"key1" : "value1","key2" : "value2"})
db.tableName.remove({"_id" : "needRemoveId"})
```
3. es
-- 查看索引
```
curl -XGET 127.0.0.1:9200/_cat/indices
```
-- 查结构
```
curl -XGET 127.0.0.1:9200/idx_cwms?pretty
```
-- 删除超时
```
&wait_for_completion=false
```
-- 查询语句是否结束  
```
curl -XGET 127.0.0.1:9200/_tasks/MVnZKEWySweaodICKYuOQA:138?pretty
```
-- 查询报错超出最大值
```
curl -XPUT http://127.0.0.1:9200/idx_cwms/_settings -d '{ "index" : { "max_result_window" : 1000000}}'
```
-- Tips
1. 时间默认为ms
2. 删除结束后要重启els

## 前端调用摄像头录像并保存到本地

```
<!DOCTYPE html>
<html>

<head>
    <title>video recoder</title>
<!--    <script src="fileSaver.js"></script>-->
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
</head>
<style>
    body{
        background-color:#EFEDEF;
    }
</style>
<body>
<article style="border:1px solid white;width:400px;height:400px;margin:0 auto;background-color:white;">

    <section class="experiment" style="width:320px; height:240px;border:1px solid green; margin:50px auto;">
        <div id="videos-container" style="width:320px; height:240px;">
        </div>
    </section>
    <section class="experiment" style="text-align:center;border:none; margin-top:20px;">
        <button id="openCamera" hidden>打开摄像头</button>
        <button id="start-recording">开始录制</button>
        <button id="save-recording" disabled>保存</button>
<!--        <a href="javascript:void(0)" onclick="send()">发送</a>-->
    </section>
</article>
<script>
    var mediaStream;
    var recorderFile;
    var stopRecordCallback;
    var openBtn = document.getElementById("openCamera");
    var startBtn = document.getElementById("start-recording");
    var saveBtn = document.getElementById("save-recording");
    openBtn.onclick = function() {
        this.disabled = true;
        startBtn.disabled=false;
        openCamera();
    };

    startBtn.onclick = function() {
		this.disabled = true;
		openCamera();
		setTimeout(function(){
            startRecord();
        }, 4000);
		setTimeout(function(){
			saveBtn.disabled=false;
		}, 5000);
    };

    saveBtn.onclick = function() {
		setTimeout(function(){
            // 结束
            stopRecord(function() {
                alert("录制成功!");
				startBtn.disabled=false;
				saveBtn.disabled=true;
				// saver();
                send();
            });
        }, 1000);
        // alert('Drop WebM file on Chrome or Firefox. Both can play entire file. VLC player or other players may not work.');
    };

    var mediaRecorder;
    var videosContainer = document.getElementById('videos-container');

	function sleep(time) {
	   return new Promise(resolve => setTimeout(resolve, time));
	}
	
    function openCamera(){
        var len = videosContainer.childNodes.length;
        for(var i=0;i<len;i++){
            videosContainer.removeChild(videosContainer.childNodes[i]);
        }

        var video = document.createElement('video');

        var videoWidth = 320;
        var videoHeight = 240;

        video.controls = false;
        video.muted = true;
        video.width = videoWidth;
        video.height = videoHeight;
        MediaUtils.getUserMedia(true, false, function (err, stream) {
            if (err) {
                throw err;
            } else {
                // 通过 MediaRecorder 记录获取到的媒体流
                console.log();
                mediaRecorder = new MediaRecorder(stream);
                mediaStream = stream;
                var chunks = [], startTime = 0;
                video.srcObject = stream;
                video.play();

                videosContainer.appendChild(video);
                mediaRecorder.ondataavailable = function(e) {
                    mediaRecorder.blobs.push(e.data);
                    chunks.push(e.data);
                };
                mediaRecorder.blobs = [];

                mediaRecorder.onstop = function (e) {
                    recorderFile = new Blob(chunks, { 'type' : mediaRecorder.mimeType });
                    chunks = [];
                    if (null != stopRecordCallback) {
                        stopRecordCallback();
                    }
                };
            }
        });
    }

    // 停止录制
    function stopRecord(callback) {
        stopRecordCallback = callback;
        // 终止录制器
        mediaRecorder.stop();
        // 关闭媒体流
        MediaUtils.closeStream(mediaStream);
    }

    var MediaUtils = {
        /**
         * 获取用户媒体设备(处理兼容的问题)
         * @param videoEnable {boolean} - 是否启用摄像头
         * @param audioEnable {boolean} - 是否启用麦克风
         * @param callback {Function} - 处理回调
         */
        getUserMedia: function (videoEnable, audioEnable, callback) {
            navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia
                || navigator.msGetUserMedia || window.getUserMedia;
            var constraints = {video: videoEnable, audio: audioEnable};
            if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                navigator.mediaDevices.getUserMedia(constraints).then(function (stream) {
                    callback(false, stream);
                })['catch'](function(err) {
                    callback(err);
                });
            } else if (navigator.getUserMedia) {
                navigator.getUserMedia(constraints, function (stream) {
                    callback(false, stream);
                }, function (err) {
                    callback(err);
                });
            } else {
                callback(new Error('Not support userMedia'));
            }
        },

        /**
         * 关闭媒体流
         * @param stream {MediaStream} - 需要关闭的流
         */
        closeStream: function (stream) {
            if (typeof stream.stop === 'function') {
                stream.stop();
            }
            else {
                let trackList = [stream.getAudioTracks(), stream.getVideoTracks()];

                for (let i = 0; i < trackList.length; i++) {
                    let tracks = trackList[i];
                    if (tracks && tracks.length > 0) {
                        for (let j = 0; j < tracks.length; j++) {
                            let track = tracks[j];
                            if (typeof track.stop === 'function') {
                                track.stop();
                            }
                        }
                    }
                }
            }
        }
    };

    function startRecord() {
		mediaRecorder.start();
    }

    function saver(){
        var file = new File([recorderFile], 'msr-' + (new Date).toISOString().replace(/:|\./g, '-') + '.mp4', {
            type: 'video/mp4'
        });
        saveAs(file);
    }

    function send(){
        var file = new File([recorderFile], 'msr-' + (new Date).toISOString().replace(/:|\./g, '-') + '.mp4', {
            type: 'video/mp4'
        });
        var data = new FormData();
        data.append("fileName", "test");
        data.append("file", file);

        var req = new XMLHttpRequest();
        req.open("POST", "saveRecording");
        req.send(data);
    }


/*
* FileSaver.js
* A saveAs() FileSaver implementation.
*
* By Eli Grey, http://eligrey.com
*
* License : https://github.com/eligrey/FileSaver.js/blob/master/LICENSE.md (MIT)
* source  : http://purl.eligrey.com/github/FileSaver.js
*/

    // The one and only way of getting global scope in all environments
    // https://stackoverflow.com/q/3277182/1008999
    var _global = typeof window === 'object' && window.window === window
        ? window : typeof self === 'object' && self.self === self
            ? self : typeof global === 'object' && global.global === global
                ? global
                : this

    function bom (blob, opts) {
        if (typeof opts === 'undefined') opts = { autoBom: false }
        else if (typeof opts !== 'object') {
            console.warn('Deprecated: Expected third argument to be a object')
            opts = { autoBom: !opts }
        }

        // prepend BOM for UTF-8 XML and text/* types (including HTML)
        // note: your browser will automatically convert UTF-16 U+FEFF to EF BB BF
        if (opts.autoBom && /^\s*(?:text\/\S*|application\/xml|\S*\/\S*\+xml)\s*;.*charset\s*=\s*utf-8/i.test(blob.type)) {
            return new Blob([String.fromCharCode(0xFEFF), blob], { type: blob.type })
        }
        return blob
    }

    function download (url, name, opts) {
        var xhr = new XMLHttpRequest()
        xhr.open('GET', url)
        xhr.responseType = 'blob'
        xhr.onload = function () {
            saveAs(xhr.response, name, opts)
        }
        xhr.onerror = function () {
            console.error('could not download file')
        }
        xhr.send()
    }

    function corsEnabled (url) {
        var xhr = new XMLHttpRequest()
        // use sync to avoid popup blocker
        xhr.open('HEAD', url, false)
        try {
            xhr.send()
        } catch (e) {}
        return xhr.status >= 200 && xhr.status <= 299
    }

    // `a.click()` doesn't work for all browsers (#465)
    function click (node) {
        try {
            node.dispatchEvent(new MouseEvent('click'))
        } catch (e) {
            var evt = document.createEvent('MouseEvents')
            evt.initMouseEvent('click', true, true, window, 0, 0, 0, 80,
                20, false, false, false, false, 0, null)
            node.dispatchEvent(evt)
        }
    }

    // Detect WebView inside a native macOS app by ruling out all browsers
    // We just need to check for 'Safari' because all other browsers (besides Firefox) include that too
    // https://www.whatismybrowser.com/guides/the-latest-user-agent/macos
    var isMacOSWebView = _global.navigator && /Macintosh/.test(navigator.userAgent) && /AppleWebKit/.test(navigator.userAgent) && !/Safari/.test(navigator.userAgent)

    var saveAs = _global.saveAs || (
        // probably in some web worker
        (typeof window !== 'object' || window !== _global)
            ? function saveAs () { /* noop */ }

            // Use download attribute first if possible (#193 Lumia mobile) unless this is a macOS WebView
            : ('download' in HTMLAnchorElement.prototype && !isMacOSWebView)
            ? function saveAs (blob, name, opts) {
                var URL = _global.URL || _global.webkitURL
                var a = document.createElement('a')
                name = name || blob.name || 'download'

                a.download = name
                a.rel = 'noopener' // tabnabbing

                // TODO: detect chrome extensions & packaged apps
                // a.target = '_blank'

                if (typeof blob === 'string') {
                    // Support regular links
                    a.href = blob
                    if (a.origin !== location.origin) {
                        corsEnabled(a.href)
                            ? download(blob, name, opts)
                            : click(a, a.target = '_blank')
                    } else {
                        click(a)
                    }
                } else {
                    // Support blobs
                    a.href = URL.createObjectURL(blob)
                    setTimeout(function () { URL.revokeObjectURL(a.href) }, 4E4) // 40s
                    setTimeout(function () { click(a) }, 0)
                }
            }

            // Use msSaveOrOpenBlob as a second approach
            : 'msSaveOrOpenBlob' in navigator
                ? function saveAs (blob, name, opts) {
                    name = name || blob.name || 'download'

                    if (typeof blob === 'string') {
                        if (corsEnabled(blob)) {
                            download(blob, name, opts)
                        } else {
                            var a = document.createElement('a')
                            a.href = blob
                            a.target = '_blank'
                            setTimeout(function () { click(a) })
                        }
                    } else {
                        navigator.msSaveOrOpenBlob(bom(blob, opts), name)
                    }
                }

                // Fallback to using FileReader and a popup
                : function saveAs (blob, name, opts, popup) {
                    // Open a popup immediately do go around popup blocker
                    // Mostly only available on user interaction and the fileReader is async so...
                    popup = popup || open('', '_blank')
                    if (popup) {
                        popup.document.title =
                            popup.document.body.innerText = 'downloading...'
                    }

                    if (typeof blob === 'string') return download(blob, name, opts)

                    var force = blob.type === 'application/octet-stream'
                    var isSafari = /constructor/i.test(_global.HTMLElement) || _global.safari
                    var isChromeIOS = /CriOS\/[\d]+/.test(navigator.userAgent)

                    if ((isChromeIOS || (force && isSafari) || isMacOSWebView) && typeof FileReader !== 'undefined') {
                        // Safari doesn't allow downloading of blob URLs
                        var reader = new FileReader()
                        reader.onloadend = function () {
                            var url = reader.result
                            url = isChromeIOS ? url : url.replace(/^data:[^;]*;/, 'data:attachment/file;')
                            if (popup) popup.location.href = url
                            else location = url
                            popup = null // reverse-tabnabbing #460
                        }
                        reader.readAsDataURL(blob)
                    } else {
                        var URL = _global.URL || _global.webkitURL
                        var url = URL.createObjectURL(blob)
                        if (popup) popup.location = url
                        else location.href = url
                        popup = null // reverse-tabnabbing #460
                        setTimeout(function () { URL.revokeObjectURL(url) }, 4E4) // 40s
                    }
                }
    )

    _global.saveAs = saveAs.saveAs = saveAs

    if (typeof module !== 'undefined') {
        module.exports = saveAs;
    }
/*
* FileSaver.js
* A saveAs() FileSaver implementation.
*
* By Eli Grey, http://eligrey.com
*
* License : https://github.com/eligrey/FileSaver.js/blob/master/LICENSE.md (MIT)
* source  : http://purl.eligrey.com/github/FileSaver.js
*/

</script>

</body>

</html>
```

## 项目添加全局日志链路
dubbo文件中加入filter

```
<dubbo:provider timeout="5000" filter="dubboFilter"/>
```

log输出格式中加入

```
%X{traceId}
```

resources/META-INF/dubbo  文件夹下加入文件

com.alibaba.dubbo.rpc.Filter

```
dubboFilter=com.***.framework.web.interceptor.DubboFilter
```

在对应位置加入DubboFilter

```
package com.***.framework.web.interceptor;

import com.alibaba.dubbo.rpc.*;
import com.***.util.StringUtils;
import org.slf4j.MDC;

public class DubboFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        RpcContext context = RpcContext.getContext();
        String traceId = context.getAttachment("traceId");
        if(StringUtils.isNotBlank(traceId)){
            TraceIdUtil.setTraceId(traceId);
            MDC.put("traceId",traceId);
        }else {
            System.err.println("something must be wrong ========== "  +  invocation.getMethodName());
        }
        Result result = invoker.invoke(invocation);
        return result;
    }
}
```

相关工具类

```
TraceIdUtil
```

```
package com.***.framework.web.interceptor;

import com.***.util.StringUtils;
import org.slf4j.MDC;

import java.util.UUID;

public class TraceIdUtil {
    private static final String TRACE_ID = "traceId";

    private static final String DEFAULT_TRACE_ID = "0";

    public static void setTraceId(String traceId) {
        traceId = StringUtils.isBlank(traceId) ? DEFAULT_TRACE_ID : traceId;
        MDC.put(TRACE_ID,traceId);
    }

    public static String getTraceId() {
        String traceId = MDC.get(TRACE_ID);
        return StringUtils.isBlank(traceId) ? DEFAULT_TRACE_ID : traceId;
    }

    public static boolean defaultTraceId(String traceId) {
        return DEFAULT_TRACE_ID.equals(traceId);
    }

    public static String genTraceId() {
        return UUID.randomUUID().toString().replace("-", "");
    }
}
```

坑点： 不可直接在有多个调用的地方加入MDC，以及RpcContext，支持的日志工具类有所受限支持org.apache.commons.logging.Log

因为他们父子线程之间的信息并不是共享的，因此可以使用traceUtil先生成一个

但是不知到这个线程是否安全，有待观察

## ES语句食用指北
最近突然要写ES语句了，不会写啊摔。
就觉得应该有其他语句转ES的工具网站。
于是我就找着了。
http://www.ischoolbar.com/EsParser/

Mysql语句：
select * from A where time between "100"  and "200" and customerId ="hello" or (customerId = "world" and world = 100);
对应ES语句：
{
	"query": {
		"bool": {
			"filter": [{
				"range": {
					"time": {
						"gte": "100",
						"lte": "200"
					}
				}
			}, {
				"bool": {
					"must": [{
						"bool": {
							"should": [{
								"match_phrase": {
									"customerId": {
										"query": "hello"
									}
								}
							}]
						}
					}]
				}
			}, {
				"bool": {
					"must": [{
						"match_phrase": {
							"customerId": {
								"query": "world"
							}
						}
					}]
				}
			}, {
				"bool": {
					"must": [{
						"match_phrase": {
							"world": {
								"query": "100"
							}
						}
					}]
				}
			}]
		}
	},
	"_source": {
		"include": ["*"]
	}
}

总结：
嘛先梳理下结构
├──query          查询
├──bool           布尔值（filter 与 _source）
├────filter       过滤
└────────range    范围时间
├────────bool     
└────────should   存在？
├────────bool     
└────────must     直等于
└──_source

感觉sql语句解析的并不正确未体现出or的效果
去打牌了，今天先记到这儿，坑下次再补上

## NGROK食用指北   
windows  powershell   
cd 进入ngrok.exe目录   

--  不确认此步骤是否必要  但我成功前确实执行过一遍    
./ngrok authtoken  官网查下token   
   
--  80换为本机需要的端口   
./ngrok http 80   

-- 官网相关
https://dashboard.ngrok.com/get-started/setup

## HI-YRYC

You can use the [editor on GitHub](https://github.com/HI-YRYC/HI-YRYC.github.io/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.
              
## Hello,World!
### Hello,YRYC!

The first blog on github.

**这是加粗的文字**
*这是倾斜的文字*`
***这是斜体加粗的文字***
~~这是加删除线的文字~~

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/HI-YRYC/HI-YRYC.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
