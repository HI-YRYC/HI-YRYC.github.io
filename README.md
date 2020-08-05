## HI-YRYC

You can use the [editor on GitHub](https://github.com/HI-YRYC/HI-YRYC.github.io/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

## LEARNING Chapter 1  
### What are HTML、HTML5、XHTML、CSS、SQL、JavaScript、PHP、ASP.NET、Web Services in website building technology?

                HTTP REQUEST                QUERY DATA

              ------------------>       ------------------>

  Computer User         Internet              Server                     Database

              <------------------       <------------------

                HTTP RESPONSE               RETURN DATA
                

    1.HTML:标记语言
      eg.[/img]图片地址 [/url]超链接 [/del]删除效果
      With out using CSS,it will display according to the browser's default style,like lists,pictures,urls,input boxes,bottons,etc.
      The style is ugly.  
    2.CSS:CSS has the same format as(property:value)
      eg.元素{
        内置
      }
      .zu-top{  bjb54sfdsf15sdfds.z.css:1
          position:fixed;   //It's position is fixed.
          left:0px;
          top:0px;          //The navigation bar should be close to the top left corner of the window.
          z-index:20;
          width:100%;       
          height:45px;      //The width and height of this navigation bar. 
          background::-moz-linear-gradient(center top,rgb(8,110,213),rgb(5,93,181)) repeat-x scroll 0%  0%  rgb(7,103,200);
                            //The background of this navigation bar is a gradient blue.
          border-bottom:1px solid rgb(4,78,151);
          box-shadow:0px 1px 2px rgba(0,0,0,0.25),0px 1px 0px rgba(255,255,255,0.15) inset;
      }
      With CSS the style will be more neatly.
      3.HTML5 and XHTML
      HTML5 is a version of HTML.
      XHTML is a composion from XML and HTML.Strict syntax requirments,some syntax differences from HTML,for XML compatibility.
      4.JavaScript
      By js we can make dynamic pages.We write js save as ***.js,in HTML we can combine it with <script>.
      When mouse hover on the label it will create a new little <div> window.
      Use js request to server,we get data in this  little <div> window.This also called AJAX.Without refreshing the page,just refresh a part of page.
      /*
      **Above are front-end,next are back-end.**
      */
      5.Web Server and Web Services
      Browser sends request to server.They all obey HTTP rules.
      eg.200 means OK.404 means Not found.
      There are so many things in HTTP RESPONSE.Content-type includes(txt,cartoon,pictures,mp3...)
      Web Server return HTML codes by url.It can be your PC or any other electronic equipments.It also can provide cache and load balancing.There is a lot open source Web Servers such as Apache,NgnixIIS.Also,you can create a new one by Node.js.Cause Web Server needs good performace,generally written in C,C++ and Java.
      6.PHP,Server Scripts,Web Framework
      Server Scripts return different pages by query data from database.
      PHP is one of Server Scripts.
      Web Framework can defend some attacks,realize SSO,handle login status and user settings by cookie,generate web templates.
              
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
