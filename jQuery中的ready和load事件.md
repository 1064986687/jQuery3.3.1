 >   jQuery中有3种针对文档加载的方式：
    1、$(document).ready(function(){
        //...code
    })
    2、$(function(){
        //...code
    })
    3、$(document).load(function(){
        //...code
    })
    其中ready和load的区别是：
    从文档加载的步骤来看，ready先执行，load后执行
    1、解析HTML结构
    2、加载外部脚本和样式表文件
    3、解析并执行脚本代码
    4、构建HTML DOM模型 //ready
    5、加载图片视频等外部文件
    6、页面加载完成 //load

>   结论：
    ready和load的区别就在于资源文件的加载，ready构建了基本的DOM结构，所以对代码来说应该加载越快越好。不需要等到图片等资源都加载完后才去处理框架的加载，图片等资源过多load事件就会迟迟不触发。
    看看jQuery中怎么写的：

    `
    jQuery.readyException = function( error ) {
        window.setTimeout( function() {
            throw error;
        } );
    };
    // The deferred used on DOM ready
    var readyList = jQuery.Deferred();

    jQuery.fn.ready = function( fn ) {

        readyList
            .then( fn )

            // Wrap jQuery.readyException in a function so that the lookup
            // happens at the time of error handling instead of callback
            // registration.
            .catch( function( error ) {
                jQuery.readyException( error );
            } );

        return this;
    };

    jQuery.extend( {

        // Is the DOM ready to be used? Set to true once it occurs.
        isReady: false,

        // A counter to track how many items to wait for before
        // the ready event fires. See #6781
        readyWait: 1,

        // Handle when the DOM is ready
        ready: function( wait ) {

            // Abort if there are pending holds or we're already ready
            if ( wait === true ? --jQuery.readyWait : jQuery.isReady ) {
                return;
            }

            // Remember that the DOM is ready
            jQuery.isReady = true;

            // If a normal DOM Ready event fired, decrement, and wait if need be
            if ( wait !== true && --jQuery.readyWait > 0 ) {
                return;
            }

            // If there are functions bound, to execute
            readyList.resolveWith( document, [ jQuery ] );
        }
    } );

    jQuery.ready.then = readyList.then;

    // The ready event handler and self cleanup method
    function completed() {
        document.removeEventListener( "DOMContentLoaded", completed );
        window.removeEventListener( "load", completed );
        jQuery.ready();
    }

    // Catch cases where $(document).ready() is called
    // after the browser event has already occurred.
    // Support: IE <=9 - 10 only
    // Older IE sometimes signals "interactive" too soon
    if ( document.readyState === "complete" ||
        ( document.readyState !== "loading" && !document.documentElement.doScroll ) ) {

        // Handle it asynchronously to allow scripts the opportunity to delay ready
        window.setTimeout( jQuery.ready );

    } else {

        // Use the handy event callback
        document.addEventListener( "DOMContentLoaded", completed );

        // A fallback to window.onload, that will always work
        window.addEventListener( "load", completed );
    }
    `