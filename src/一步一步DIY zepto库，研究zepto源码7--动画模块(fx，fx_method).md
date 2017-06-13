---
title: 一步一步DIY zepto库，研究zepto源码7--动画模块(fx fx_method)
tags:    
- FE
- zepto
- 源码
- js原生实现库    
categories: 前端技术
---
> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，对应文件夹v0.7.1。
> https://github.com/zrysmt/DIY-zepto

注：要在[github源代码中](https://github.com/madrobby/zepto)自己编译的话，要在基础包`命令：npm run dist`上要进行扩展了，输入命令：

```bash
MODULES="zepto event fx fx_methods" npm run dist
# on Windows
> SET MODULES=zepto event fx fx_methods
> npm run dist
```
# 1.示例Demo
```javascript
var div1 = $('#foo1');
div1.animate({
        "width": "300px",
        "height": "300px"
    },
    'slow', 'ease-in-out',
    function() {
        console.log('div1 animate callback');
        // div2.hide('slow',function(){
        div2.fadeOut('slow', function() {
            console.log('div2 animate callback');
        });
    }, '2000');
```
# 2.fx
zepto的动画采用的是CSS3的动画/过渡，未做兼容。
核心方法是`$.fn.animate = function(properties, duration, ease, callback, delay) `，实际上的处理逻辑是`$.fn.anim = function(properties, duration, ease, callback, delay)`
其本质就是设置好css3属性对象，然后用`this.css(cssValues)`方法是css3动画起作用。浏览器不支持的动画使用setTimeout
```javascript
// duration为0，即浏览器不支持动画的情况，直接执行动画结束，执行回调。
if (duration <= 0) setTimeout(function() {
   that.each(function() { wrappedCallback.call(this); });
}, 0);
```
整体的源代码和注释放在下面
```javascript
var Fx = function($) {
    var prefix = '',
        eventPrefix,
        vendors = { Webkit: 'webkit', Moz: '', O: 'o' },
        testEl = document.createElement('div'),
        supportedTransforms = /^((translate|rotate|scale)(X|Y|Z|3d)?|matrix(3d)?|perspective|skew(X|Y)?)$/i,
        transform,
        transitionProperty, transitionDuration, transitionTiming, transitionDelay, //过渡
        animationName, animationDuration, animationTiming, animationDelay, //动画
        cssReset = {};

    //将驼峰字符串转成css属性，如aB-->a-b
    function dasherize(str) {
        return str.replace(/([a-z])([A-Z])/, '$1-$2').toLowerCase();
    }
    //修正事件名
    function normalizeEvent(name) {
        return eventPrefix ? eventPrefix + name : name.toLowerCase();
    }

    /**
     * 根据浏览器内核，设置CSS前缀，事件前缀
     * 如css：-webkit-  event:webkit
     * 为prefix和eventPrefix赋值
     */
    if (testEl.style.transform === undefined) {
        $.each(vendors, function(vendor, event) {
            if (testEl.style[vendor + 'TransitionProperty'] !== undefined) {
                prefix = '-' + vendor.toLowerCase() + '-';
                eventPrefix = event;
                return false;
            }
        });
    }

    transform = prefix + 'transform';
    //均为空''
    cssReset[transitionProperty = prefix + 'transition-property'] =
        cssReset[transitionDuration = prefix + 'transition-duration'] =
        cssReset[transitionDelay = prefix + 'transition-delay'] =
        cssReset[transitionTiming = prefix + 'transition-timing-function'] =
        cssReset[animationName = prefix + 'animation-name'] =
        cssReset[animationDuration = prefix + 'animation-duration'] =
        cssReset[animationDelay = prefix + 'animation-delay'] =
        cssReset[animationTiming = prefix + 'animation-timing-function'] = '';
    /**
     * 动画常量数据源
     * @type {{off: boolean, speeds: {_default: number, fast: number, slow: number}, cssPrefix: string, transitionEnd: *, animationEnd: *}}
     */
    $.fx = {
        off: (eventPrefix === undefined && testEl.style.transitionProperty === undefined), //能力检测是否支持动画，具体检测是否支持过渡，支持过渡事件
        speeds: { _default: 400, fast: 200, slow: 600 },
        cssPrefix: prefix, //css 前缀  如-webkit-
        transitionEnd: normalizeEvent('TransitionEnd'), //过渡结束事件
        animationEnd: normalizeEvent('AnimationEnd') //动画播放结束事件
    };
    /**
     * [animate 自定义动画]
     * @param  {[Object]}   properties [属性变化成，如{"width":"300px"}]
     * @param  {[type]}   duration   [速度 如slow或者一个数字]
     * @param  {[type]}   ease       [变化的速率ease、linear、ease-in / ease-out、ease-in-out
cubic-bezier]
     * @param  {Function} callback   [回调函数]
     * @param  {[type]}   delay      [延迟时间]
     */
    $.fn.animate = function(properties, duration, ease, callback, delay) {
        //参数处理
        if ($.isFunction(duration)) //传参为function(properties,callback)
            callback = duration, ease = undefined, duration = undefined;
        if ($.isFunction(ease)) //传参为function(properties,duration，callback)
            callback = ease, ease = undefined;
        if ($.isPlainObject(duration)) //传参为function(properties,｛｝)
            ease = duration.easing, callback = duration.complete, delay = duration.delay, duration = duration.duration
            //duration参数处理
        if (duration) duration = (typeof duration == 'number' ? duration :
            ($.fx.speeds[duration] || $.fx.speeds._default)) / 1000;
        if (delay) delay = parseFloat(delay) / 1000;
        return this.anim(properties, duration, ease, callback, delay);
    };
    $.fn.anim = function(properties, duration, ease, callback, delay) {
        var key, cssValues = {},
            cssProperties, transforms = '',
            that = this,
            wrappedCallback, endEvent = $.fx.transitionEnd,
            fired = false;
        //修正好时间
        if (duration === undefined) duration = $.fx.speeds._default / 1000;
        if (delay === undefined) delay = 0;
        if ($.fx.off) duration = 0; //如果浏览器不支持动画，持续时间设为0，直接跳动画结束

        if (typeof properties == 'string') {
            // keyframe animation
            cssValues[animationName] = properties; //properties是动画名
            cssValues[animationDuration] = duration + 's';
            cssValues[animationDelay] = delay + 's';
            cssValues[animationTiming] = (ease || 'linear');
            endEvent = $.fx.animationEnd;
        } else { //properties 是样式集对象
            cssProperties = [];
            // CSS transitions
            for (key in properties) {
                //是这些属性^((translate|rotate|scale)(X|Y|Z|3d)?|matrix(3d)?|perspective|skew(X|Y)?)
                if (supportedTransforms.test(key)) {
                    transforms += key + '(' + properties[key] + ') '; //拼凑成变形方法
                } else {
                    cssValues[key] = properties[key], cssProperties.push(dasherize(key));
                }
            }
            if (transforms) cssValues[transform] = transforms, cssProperties.push(transform);
            if (duration > 0 && typeof properties === 'object') {
                cssValues[transitionProperty] = cssProperties.join(', ');
                cssValues[transitionDuration] = duration + 's';
                cssValues[transitionDelay] = delay + 's';
                cssValues[transitionTiming] = (ease || 'linear');
            }
        }
        //动画完成后的响应函数
        wrappedCallback = function(event) {
            if (typeof event !== 'undefined') {
                if (event.target !== event.currentTarget) return; // makes sure the event didn't bubble from "below"
                $(event.target).unbind(endEvent, wrappedCallback);
            } else {
                $(this).unbind(endEvent, wrappedCallback); // triggered by setTimeout
            }

            fired = true;
            $(this).css(cssReset);

            callback && callback.call(this);
        };
        //处理动画结束事件
        if (duration > 0) {
            //绑定动画结束事件
            this.bind(endEvent, wrappedCallback);
            // transitionEnd is not always firing on older Android phones
            // so make sure it gets fired
            //延时ms后执行动画，注意这里加了25ms，保持endEvent，动画先执行完。
            //绑定过事件还做延时处理，是transitionEnd在older Android phones不一定触发    
            setTimeout(function() {
                if (fired) return;
                wrappedCallback.call(that);
            }, ((duration + delay) * 1000) + 25);
        }

        // trigger page reflow so new elements can animate
        //主动触发页面回流，刷新DOM，让接下来设置的动画可以正确播放
        //更改 offsetTop、offsetLeft、 offsetWidth、offsetHeight；scrollTop、scrollLeft、
        //scrollWidth、scrollHeight；clientTop、clientLeft、clientWidth、clientHeight；getComputedStyle() 、
        //currentStyle()。这些都会触发回流。回流导致DOM重新渲染，平时要尽可能避免，
        //但这里，为了动画即时生效播放，则主动触发回流，刷新DOM

        this.size() && this.get(0).clientLeft;

        //设置样式，启动动画
        this.css(cssValues);

        // duration为0，即浏览器不支持动画的情况，直接执行动画结束，执行回调。
        if (duration <= 0) setTimeout(function() {
            that.each(function() { wrappedCallback.call(this); });
        }, 0);

        return this;
    };

    testEl = null;
};
export default Fx;
```

# 3.fx_methods
源码中的`fx_methods`（fx_methods.js中）方法，说白了就是利用上面的`fx.js`文件下的`$.fn.animate`函数提供便捷的方法

整体源码和注释放在这里

```javascript
var Fx_methods = function($) {
    var document = window.document,
        docElem = document.documentElement,
        origShow = $.fn.show,
        origHide = $.fn.hide,
        origToggle = $.fn.toggle;

    function anim(el, speed, opacity, scale, callback) {
        if (typeof speed == 'function' && !callback) callback = speed, speed = undefined;
        var props = { opacity: opacity };
        if (scale) {
            props.scale = scale;
            el.css($.fx.cssPrefix + 'transform-origin', '0 0'); //设置变形原点
        }
        return el.animate(props, speed, null, callback); //不支持速率变化
    }

    function hide(el, speed, scale, callback) {
        //$(dom).animate({opacity: 0, '-webkit-transform-origin': '0px 0px 0px', '-webkit-transform': 'scale(0, 0)' },800)
        //设置了变形原点，缩放为0，它就会缩到左上角再透明
        return anim(el, speed, 0, scale, function() {
            origHide.call($(this));
            callback && callback.call(this);
        });
    }

    $.fn.show = function(speed, callback) {
        origShow.call(this);
        if (speed === undefined) {
            speed = 0;
        } else {
            this.css('opacity', 0);
        }
        return anim(this, speed, 1, '1,1', callback);
    };

    $.fn.hide = function(speed, callback) {
        if (speed === undefined) {
            return origHide.call(this);
        } else {
            return hide(this, speed, '0,0', callback);
        }
    };

    $.fn.toggle = function(speed, callback) {
        if (speed === undefined || typeof speed == 'boolean') {
            return origToggle.call(this, speed);
        } else {
            return this.each(function() {
                var el = $(this);
                el[el.css('display') == 'none' ? 'show' : 'hide'](speed, callback);
            });

        }
    };

    $.fn.fadeTo = function(speed, opacity, callback) {
        return anim(this, speed, opacity, null, callback);
    };

    $.fn.fadeIn = function(speed, callback) {
        var target = this.css('opacity');
        if (target > 0){
         this.css('opacity', 0);
        }
        else {
            target = 1;
        }
        return origShow.call(this).fadeTo(speed, target, callback);
    };

    $.fn.fadeOut = function(speed, callback) {
        return hide(this, speed, null, callback);
    };

    $.fn.fadeToggle = function(speed, callback) {
        return this.each(function() {
            var el = $(this);
            el[
                (el.css('opacity') == 0 || el.css('display') == 'none') ? 'fadeIn' : 'fadeOut'
            ](speed, callback);
        });
    };
};

export default Fx_methods;
```
> 全部代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，本博文对应文件夹v0.7.x。
> https://github.com/zrysmt/DIY-zepto

参考阅读：
- [Zepto源码分析-动画(fx fx_method)模块](http://www.cnblogs.com/mominger/p/4538685.html)