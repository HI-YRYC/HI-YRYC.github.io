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
