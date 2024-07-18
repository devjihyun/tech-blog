# 이미지 레이지 로드 (Image lazy load)

## 개념

이미지 레이지 로드 기법은 웹 페이지 로딩 시 초기 화면에 필요한 이미지들만 먼저 로드하고, 사용자가 스크롤하여 해당 이미지들이 필요한 순간에 나머지 이미지들을 로드하는 기술입니다. 이를 통해 페이지 로딩 속도를 개선하고 사용자 경험을 향상시킬 수 있습니다.

## 사례

### loading 속성 사용
이 속성은 브라우저가 해당 이미지를 뷰포트에 근접했을 때만 로드하도록 지시합니다. 이는 CSS와 JavaScript를 이용한 전통적인 방법보다 간단하고 성능에 유리합니다.

```html
<!-- 화면 근처를 스크롤할 때 이미지를 레이지 로드 -->
<img src="example.jpg" loading="lazy" alt=".."/>

<!-- 이미지 바로 로드 -->
<img src="example.jpg" loading="eager" alt=".."/>

<!-- 브라우저가 이미지 레이지 로드 여부를 결정 -->
<img src="example.jpg" loading="auto" alt=".."/>

<!-- <picture>의 레이지 로드 이미지 -->
<picture>
    <source media="(min-width: 40em)" srcset="big.jpg 1x, big-hd.jpg 2x">
    <source srcset="small.jpg 1x, small-hd.jpg 2x">
    <img src="fallback.jpg" loading="lazy">
</picture>

<!-- srcset이 지정된 이미지를 레이지 로드 -->
<img src="small.jpg" srcset="large.jpg 1024w, medium.jpg 640w, small.jpg 320w" sizes="(min-width: 36em) 33.3vw, 100vw" alt="A rad wolf" loading="lazy">

<!-- 화면 근처를 스크롤할 때 iframe 이미지를 레이지 로드 -->
<iframe src="video-player.html" loading="lazy"></iframe>
```

[references]  
[Lazy load images and  &lt;iframe&gt;  elements](https://web.dev/learn/performance/lazy-load-images-and-iframe-elements)  
[A Deep Dive Into Native Lazy-Loading For Images And Frames](https://css-tricks.com/a-deep-dive-into-native-lazy-loading-for-images-and-frames)

[sites]  
Apple, Naver

### Intersection Observer API를 활용
이 API는 이미지가 뷰포트에 들어오는 것을 감지하여 로딩을 트리거합니다.  예시 코드는 다음과 같습니다 :

```javascript
<img data-src="example.jpg" alt="Example Image" class="lazy">

document.addEventListener("DOMContentLoaded", function() {
	let lazyImages = [].slice.call(document.querySelectorAll("img.lazy"));
	if ("IntersectionObserver" in window) {
		let lazyImageObserver = new IntersectionObserver(function(entries, observer) {
			entries.forEach(function(entry) {
				if (entry.isIntersecting) {
					let lazyImage = entry.target;
					lazyImage.src = lazyImage.dataset.src;
					lazyImage.classList.remove("lazy");
					lazyImageObserver.unobserve(lazyImage);
				}
			});
		});
		lazyImages.forEach(function(lazyImage) {
			lazyImageObserver.observe(lazyImage);
		});
	}
});
```

[references]   
[Five Ways to Lazy Load Images for Better Website Performance](https://www.sitepoint.com/five-techniques-lazy-load-images-website-performance/)  
[Lazy load offscreen images with lazysizes](https://web.dev/articles/codelab-use-lazysizes-to-lazyload-images)


### Event Handlers (Scroll, Resize, Orientation Change)
Javascript Scroll, Resize, Orientation Change 이벤트를 활용하여 레이지 로드를 구현하는 방법.

이 방법은 스크롤, 크기 조정, 방향 변경 이벤트에 event listeners를 붙여 이미지를 로드해야 하는 시점을 결정하는 것입니다. 이벤트가 트리거되면 코드는 해당 이미지가 뷰포트에 있는지 확인한 후 로드합니다.   
이 방법은 Intersection Observer API 보다 효율성은 떨어지지만 오래된 브라우저에서는 폴백(fallback)으로 사용할 수 있습니다.

```javascript
<img data-src="image.jpg" alt="Lazy Load Example" class="lazy">

function lazyLoad() {
    document.querySelectorAll('img[data-src]').forEach(img => {
        if (img.getBoundingClientRect().top < window.innerHeight && img.getBoundingClientRect().bottom > 0) {
            img.src = img.dataset.src;
            img.removeAttribute('data-src');
        }
    });
}

window.addEventListener('scroll', lazyLoad);
window.addEventListener('resize', lazyLoad);
window.addEventListener('orientationchange', lazyLoad);
```

[references]   
[Five Ways to Lazy Load Images for Better Website Performance](https://www.sitepoint.com/five-techniques-lazy-load-images-website-performance/)  
[Lazy load offscreen images with lazysizes](https://web.dev/articles/codelab-use-lazysizes-to-lazyload-images)

[sites]  
eBay, LinkedIn

### Third-Party Libraries
몇몇 라이브러리들은 레이지 로드 구현을 단순화하고 추가적인 기능들을 제공합니다. 몇몇 인기 있는 라이브러리는 다음과 같습니다 :

- Vanilla-lazyload
```javascript
new LazyLoad({
	elements_selector: ".lazy"
});
```

- Lozad.js
```javascript
const observer = lozad();
observer.observe();
```

- LazySizes
```html
<img data-src="image.jpg" class="lazyload">
<script src="lazysizes.min.js" async=""></script>
```


### 반응형 이미지에 'srcset'과 'size' 속성 적용
lazy loading 기법과 함께 'srcset'과 'size' 속성을 사용하면 장치의 화면 크기와 해상도에 따라 적절한 크기의 이미지를 로드할 수 있습니다.

```html
<img
    src="small.jpg"
    data-src="large.jpg"
    data-srcset="small.jpg 300w, medium.jpg 600w, large.jpg 1200w"
    sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
    class="lazyload">

<script src="lazysizes.min.js" async=""></script>
```

[sites]  
https://web.dev/?hl=ko
https://css-tricks.com/


### CSS Background Images 사용하기
레이지 로딩 배경 이미지는 요소가 뷰포트에 들어올 때 클래스를 적용하는 것을 포함합니다. 자바스크립트를 사용하여 배경 이미지의 로딩을 트리거하는 클래스를 추가할 수 있습니다.


```javascript
<div class="lazy-background" data-bg="url('background.jpg')"></div>

document.addEventListener('DOMContentLoaded', function() {
    let lazyBackgrounds = [].slice.call(document.querySelectorAll(".lazy-background"));
        if ("IntersectionObserver" in window) {
            let lazyBackgroundObserver = new IntersectionObserver(function(entries, observer) {
                entries.forEach(function(entry) {
                    if (entry.isIntersecting) {
                        entry.target.style.backgroundImage = entry.target.dataset.bg;
                        lazyBackgroundObserver.unobserve(entry.target);
                    }
                });
            });

            lazyBackgrounds.forEach(function(lazyBackground) {
            lazyBackgroundObserver.observe(lazyBackground);
        });
    }
});
```



[references]   
[lazy-loading-images](https://web.dev/articles/lazy-loading-images)  
[Lazy_loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Lazy_loading)  
[optimizing-website-performance-harnessing-the-power-of-image-lazy-loading](https://www.catchpoint.com/blog/optimizing-website-performance-harnessing-the-power-of-image-lazy-loading)
[lazy-loading](https://addyosmani.com/blog/lazy-loading/)  
[hybrid-lazy-loading-progressive-migration-native](https://www.smashingmagazine.com/2019/05/hybrid-lazy-loading-progressive-migration-native/)
