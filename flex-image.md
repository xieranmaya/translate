## Pure CSS implementation of Google Photos / 500px image layout

For sharing the secrets of getting most perfect effect in every step to the end, this can be a very long post. 
So you guys can jump to the part of code, demo and HOWs, which follow the last main title, if you don’t want to read it from the beginning. Or you can just check the source code of the bellow link.

But my suggestion is to read it from the beginning, because it will help you understand the HOWs better with the information provided in the former part, which mainly solves the problem of some corner cases.

First, let’s take a look at the result to make you more interested, resize or zoom the page to check the dynamic effect.

https://xieranmaya.github.io/images/cats/cats.html

![image](https://cloud.githubusercontent.com/assets/2993947/14738104/5e28601c-08b2-11e6-8262-5351a575f3fb.png)

## Besides, to show the source code, jsbin.com is used in some demos that we will talk about later. But the jsbin.com sometimes may not work. If that happens, just rewrite the source code in the edit box without changing the meaning. For example, press the “enter” button in the last line.

### Ok, let’s start.

Firstly, let’s see the differences between three common images layout:

1. [Pinterest.com](https://www.pinterest.com/)：

    * This is a common layout with equal width, which all images share a same width, we often call it waterfall layout.
    * Due to the stretching of images in same proportion and the same width of each, all images have different heights.
    * The disadvantages of this layout is that some images might be missed by readers, because every image has different height thus start from different top position and is not displayed in common browsing order, plus people usually scan things in horizontal direction.
    * The image displaying wall usually has uneven bottom.
    * Though the bottom can not be entirely even, JS is applied to make it with reordering the photos. So JS must be involved when the page is reflowed, resized or zoomed in this layout.

2. Images layout in which image has unequal width and height with Google Photos as representation, such else as 500px and tuchong.com.

    * Images are stretched in the same proportion too.
    * The display area is fully occupied by every row of images in horizontal direction, leaving no extra blank.
    * Due to the above two rules, every image in the same row has different height, or the second rule will be broken.
    * The images are displayed in order, making it more convenient for readers to browse. This rule has to be obeyed by Apps like Google Photos because the taken time is a very important property and thus the photos should displayed in order.
    * This bottom of this layout is even.
    * In the layout of Google Photos, the images will be displayed in same row in spite of different date when there is no enough images of same date. PS. This is a secondary point we are going to talk about(which now I desided to write another post to deal with it).

3. Instangram
    
    * Which is a square layout, which is unnecessary to explain more.

The above mentioned two layouts have onething in common is that the images are stretched in same proportion with the original image. That is to say, the images are just zoomed out and in without any change of the pictures' contents. This is the tendency of common picture web App now days. In fact, few websites will adopt the method of magnified or minified in unequal proportion(which is very ugly). The last method is to clip the image to square and put it in a square box, which can be achieved by the `background-size: cover;` CSS property.

What's more, in the Pinterest layout, the display area will be smaller for the wider images. In the second layout, however, the display area will be smaller for the taller images. 

In the second layout, because different rows have different height, the higher image can have a chance to show larger when it resides in a row which has larger row height.

In whole, the image layout leaded by "Google Photos" has better effect. About how to achieve the Google Photos / 500px algorithm is JS, readers can think about it by yourself, we'll talk about CSS implementation here.

Next, let's summarize the standards of evaluating images layout based on the above analysis.

1. Whether it can output in the oringal list in sequence
2. Whether it can output in our eye's scan sequence, namely the same line-height
3. Whether the picture can display in the original proportion, or try to display as much as possible
4. Whether the picture can display it's full content, or try to display as much as possible
5. Whether the area of every image can be the same, or at least try to be the more closer(in fact, it is hard to achieve this effect in Pinterest or 500px)
6. Whether the image can be display vividly without any non-proportional stretch

When I saw "Google Photos" image layout for the first time, I guess it is much probably JS involved. Because it's impossible justified on both sides under the condition of the same picture height, and in the same separation distance between the different pictures.(If each picture have the different separation distance while it can be justified on the both sides, we can use `inline` picture and text-justify to achieve this. And it's a choice when the images are small). After my observation, every row have different height, so I am sure there must be JS involved.

But as a frontend developer, I perfer CSS to JS. However, when I saw more and mroe websites starts to use this layout, an idea occours to me: whether we can achieve this layout only by CSS instead of JS, **especially not recalculate the layout in the process of resizing/zoom**.

After some trial, I found a way to achieve similar layout only by CSS in some extent.(Only use CSS to achieve layout, what I mentioned here, referring to the subsequent steps of resizing and zooming can be stable without JS involved after rendering completed. That's to say, initial rendering can be achieved on the server, and the whole process is not involved JS. So it's natural to say that we only use CSS to achieve this layout).

### Implementation process

Next, I will introduce the the entire attempt I tried step by step to achieve this layout with pure CSS.

First of all, let's set the same height for all the images

```html
<style>
  img {
    height: 200px;
  }
</style>
<div>
   <img src="" />
   <img src="" />
   <img src="" />
   <img src="" />
   <img src="" />
</div>
```

This method can't fill the image very appropriately in the horizontal space. The `flex-grow` cames to my mind, it can make images' box to grow up to fill in the container's extra horizontal space. Naturally, whole layout become flex.


```css
div {
  display: flex;
  flex-wrap: wrap;
}
img {
  height: 200px;
  flex-grow: 1;
}
```

Set the `flex-wrap` of flex container into `wrap`. In this way, line wrap will happens if one line can not lay one more image. Every line's images will fill in the container's horinontal space due to the grow. It seems that it's almost achieved our expectation effect, but every image has been stretched in **non-proportion** so that the images are out of shape. This is easy for us, we can fix it with `object-fit: cover;`. However, part of the image will be cliped. 

Final demo: http://jsbin.com/tisaluy/1/edit?html,css,js,output

Actually, the above mentioned DOM structure can't be used in the practice

 * It does not support browser which lack the `object-fit` support, so the picture will be distorted. The reason is the images has no container, and it's not posible to use `background-size` to solve this problem.
 *By using the object-fit porperty in supported browser, part of images will be cliped( these two rules have been mentioned before).
* We can't show some information with the images together, because of the bare image tag with no container.
* In a real network environment, pictures loading are rather slow. So if we try to use the images' box to streth the layout, there must be many flash/blink or sth like FOUC, it should already occoured in demo.
 

Thus, the above mentioned layout can't be used in any kind of production situation.

Next, we change the DOM structure to this:

```html
<section>
   <div>
      <img src=""/>
   </div>
   <div>
      <img src=""/>
   </div>
   <div>
      <img src=""/>
   </div>
   <div>
      <img src=""/>
   </div>
</section>
```

We added a container for each image. Firstly, the image height is set 200px still, and every `div` will be stretched by its image child node. Next, if we set a `flex-grow: 1` for the `div`s, extra spaces in each row will be divided equally by each `div`. Then the `div` will be wider, but the image width can't fill in the div completely, it's obvious. Then, if we set the image width to 100%, the growed space will not be redivid by `div` in IE and FF.(I personally think this is a very interesting , the image stretchs div first, then the `div`'s grow stretchs the image). But in Chrome, setting 100% width for image will cause the extra space in each row to reallocate. Which in result makes every container's width to more closer. However, it’s not what we want. After several combinations of css properties, I almost did it. If we set the image's `min-width` and `max-width` to 100% at the same time, the display effect in Chrome will be the same with IE and FF. Lastly, we set the image's `object-fit` property to `cover`, and the image will fill in the container with equal proportional stretch.

This method still is not perfect just like the former one. The images height is all same, and partial image has been clip for obvious reason.

The entire demo of the above solution: http://jsbin.com/tisaluy/2/edit?html,css,output

If the image height value is set to a small value, the above layout is roughly right. There will be more images in every row, so extra space in each row will be less and divided by more images. In this way, the ratio of every container will be more close to the real image ratio. Most of images can display the main content.

The only problem occurs in the last line. When the last line only has few images, it will fill in the whole line due to `flex-grow`. However, our height is set to 200px, so the display area will show less images content. The situation will be worse when the image is higher and the display area(which is the container) is wider.

The solution for this situation is to let the last few images to not grow. We can calculate the average images in every row in case of distorting terribly. 

```css
div:nth-last-child(5),
div:nth-last-child(4),
div:nth-last-child(3),
div:nth-last-child(2),
div:nth-last-child(1){
  flex-grow: 0;
}
```

And media query is adopted here when the screen have different width, the number of elemnets in the last row will change according to the window's width:

```css
@media (max-width: 1000px) and (min-width: 900px) {
  div:nth-last-child(5),
  div:nth-last-child(4),
  div:nth-last-child(3),
  div:nth-last-child(2),
  div:nth-last-child(1){
    flex-grow: 0;
  }
}
@media (max-width: 1100px) and (min-width: 1000px) {
  div:nth-last-child(7),
  div:nth-last-child(6),
  div:nth-last-child(5),
  div:nth-last-child(4),
  div:nth-last-child(3),
  div:nth-last-child(2),
  div:nth-last-child(1){
    flex-grow: 0;
  }
}
```

Every certain width have to be many “nth-last-child” selectors so that above code is rather complicated. We can use preprocessor to loop out this code, but there are still many repetitious code generated.

Is there any way to just appoint how many elements instead of several “nth-last-child” selectors? The answer is yes. Here we should mention the “~” operator in CSS. Let’s write:

```css
div:nth-last-child(8),
div:nth-last-child(8) ~ div{
  flex-grow: 0;
}
```

First, we can select the eighth last element. Then the following node after the eighth last element can be selected by the `~` combinator. Finally, the last eight elements are selected successfully. Furthermore, we can rewrite the selector to “div:nth-last-child(9) ~ div”, and we may select the last eight elements only with one selector.

The real effect varies from the different selectors we chose in the above rear elements:

* “div:nth-last-child(1),div:nth-last-child(2),...,div:nth-last-child(n)”, this method can make sure the last N images to always selected
* “div:nth-last-child(n), div:nth-last-child(n) ~ div”, this method only make sure there must be more than N div elements, so the last N elements can be selected. The selector will no working if "nth-last-child(n)" select nothing.
* “div:nth-last-child(n+1) ~ div” , this method have to make sure there must be more than N+1 div elements, so the last N elements can be selected

Actually, the way of choosing last several images is not perfect, because it's not guaranteed that the flex items you select is in the last row. It might be only one image in the last row, and if this happens, first images in the next-to-last row will grow badly. (Because the last N images won’t grow). Or the last images in the last two rows are not up to a certain amount, and then the next-to-last row won’t grow to fill in this row. All may lead to the format disorder.

Is there any way to prevent just the last row's elements' growing? I thought for a long while, even to seek for a `last-line` or `pseudo-class`, but I failed. And then one day, I got the answer occasionally:

Adding another element to the last element as the last flex item, and sets its `flex-grow` to a extremely large value (e.g., 999999999). Then the remaining space in the last line will be taken fully up by this extra element's grow. The other elements are thus not growed, what’s more, we can achieve this by pseudo elements(But IE browser does not support flex properties on pseudo elements, so it is essential to adopt a real element placeholder):

```css
section::after {
  content: '';
  flex-grow: 999999999;
}
```

Ok, we basically solve all the problems encountered in this layout.

Demo: resize or zoom first, then observe the images in the last row: http://jsbin.com/tisaluy/3/edit?html,css,output

But there is one last question, this layout is just like the former one except for it has a container which can show some extra info for the images. If you load the layout pages online, pages will occur a severe blinking issue. Before downloading the images, we have no ideas about the width and height. It’s impossible to wait images loading completed to stretch the container. What’s worse, we need refresh pages more than ten thousand times at develop which will have a very worse effect on our developer's eyes too.

So, we have to render the images display area(i.e. its container) in advance. (Actually, almost all websites with images adopt this approche.) JS have to be applied in here. Those works can be finished on the server or by any template engines. (Below codes employ template syntax in Angular.)

Once this layout finished, all the subsequent actions(resize，zoom) won’t disorder the layout without JS involved. It did it successfully with CSS.

```html
<style>
  section {
    padding: 2px;
    display: flex;
    flex-wrap: wrap;
    &::after {
      content: '';
      flex-grow: 999999999;
    }
  }
  div {
    margin: 2px;
    position: relative;
    height: 200px;
    flex-grow: 1;
    background-color: violet;
    img {
      max-width: 100%;
      min-width: 100%;
      height: 200px;
      object-fit: cover;
      vertical-align: bottom;
    }
  }
</style>
<section>
    // The expression is used to calculate the width value when image is 200 height
    <div ng-repeat="img in imgs" style="width:{{img.width*200/img.height}}px;"></div>
</section>
```

We achieve the images layout we expected.

Demo, notice the expression in the template: http://jsbin.com/tisaluy/4/edit?html,css,output

So, what about the final effect of the layout?

Acturally I wrote code to calculate the percentage of each image shows. When images height is about 150px, one third images can show 99% content. The worst one or two case of images can display about 70% content. In average, the display percentage of all images is about 90%. The display effect will be better if the image is shorter(less height), and the display effect will be worse if the image is higher(larger height).

I gave up this solution later, so there is no need to present the detailed demo.

### You may thought you are played fool of if you don't keep reading. Because it didn’t accomplish the Google Photos images layout as you expected.

Because of the **same height in each row**, most images are not displayed completely. And this layout are totally unrelated to the marvelous Google Photos / 500px layout.

The real post starts from here, below approche came after the above method after a long while. The above content just introduce the solution for some corner cases.

We can see that the above solution doesn't make every image to display completely. If you need to accomplish 500px layout, the height of each row is most likely to not equal to each other in that layout.

For the start, I guessed CSS can dealt with nothing more than the above extent.
However, what I needed later proved me wrong:

I want to display some content in a square container, and I want the square container to always spread with the window without spare room (except the blank between the elements) regardless of the browser window’s width. At first glance, this may require the JS participation: read the browser window's width first, then calculate the size of a square container, and then render.

Open this demo to see the effect, try to resize or zoom the page: http://jsbin.com/tomipun/4/edit?html,css,output

During the process of resize / zoom the page, the square container will grow or shrink in real-time but will always be square, and its size is remaining in a specific range. It will grow to a point, later it wanes. If we only focus on one demo, it’s hard to come up with the solution to achieve the fixed aspect ratios. But if there is a square container and its side length is the half of the browser width, you guys might know the solution.

As we know, if we specify margin or padding value by percentage, the value is relative to it's parent element's width. Namely, if we give a block element 100% for its `padding-bottom` and its `height` setting to 0, the element will keep to  square and it will change with the container height. If we want to change the size of the square, we only need to change the width of the parent container, the height will change appropriately.

Check this link: 
http://jsbin.com/lixece/1/edit?html,css,output
the colored block will grow and it remains square while we resize the window. If we take browser window as a reference, this effect can be achieved with vw / vh in modern browser. If not, we can choose vertical padding to accomplish.

Then it occurs to me, if we do not set flex item's height and let it stretch by its child element, and the child element's width is 100% and padding-bottom is 100%, then both flex item the child element will keep square at the same time. As a result, the above square grid layout can be finished.

This is far away from the perfect result. If the element number in the last row differs from the number of the previous rows, color block in the last row will be larger due to growing. How to make the last row’s elements stay the same with the previous row? It doesn’t work if we use a extra element and set a large `flex-grow` value to fill the spare room in the last row. We need keep the last row's element to grow same space with the previous rows.

To be honest, the solution is simple. We just treat the last row to be not the last row, we treat the second last row as last row!

We can add some placeholder elements to let the last elements to be in the visual last row. And the placeholder elements occupied the real last row. Then change the placeholder height to 0. How many placeholders we need? The number should stay the same with the posible maximum of elements per row. Such as, the above demo has 8 extra placeholders which you can check in the source code. For better semantic meaning, we can give them special class name to act as placeholders.

In this way, the square layout which fills in the horizontal width can be accomplished(which is the previous demo,and you can check the source code or inspect to see the placeholder elements). 

Although the most advanced flexbox layout is adopted, CSS can’t accomplish the perfect layout without cropping. So, I thought the essay might stop here.

### - FAKE EOF -

But when I woke up on the morning of April 2，an idea came into my mind that since you can always keep the container to square, would it mean that you can keep it in any proportion? Certainly yes, we only have to set the padding-bottom of the child element which stretching its parent element a value we want! In this way, maybe the layout can be achieved only by CSS!

Certainly, as I mentioned before, because of the slow loading of images, this layouts should often know the width and height of the image in advance to pre-render the container, and directly put the image into it after the image is loaded or loading. 

So we still need JS or server to calculate the images' aspect ratio, and set it in the padding-bottom to ensure that the aspect ratio of the container is always the same to its internal images.

Firstly, we display all the images in the height of 200px, as shown below:

```html
<div style="display:flex;flex-wrap:wrap;">
  //The below formula calculate the value of the image width when its height is 200
  <div ng-repeat="img in imgs" style="width:{{img.width*200/img.height}}px;">
  
    //This formula set the proportion of the element and its parent element the same as the original proportion of the image. because it's vertical padding, it is the height divided by the width and then multiples 100 because the pencentage mark.
    <div style="padding-bottom:{{img.height/img.width*100}}%"></div>
    
  </div>
</div>
```

In the above layout, because of the flex-wrap, every row will be breaked and extra spare room will be left if there are too many images in one row. The aspect ratio of each container will always stays the same with the images which we will put into it later.
For demonstration purpose，I will set the size of the image a quarter of the container, it should be clear that the bottom right corner of the image is at the center of the container.

Demo: http://jsbin.com/tisaluy/5/edit?html,css,output

Next, we need to make all the elements to grow. Could we set all their `flex-grow` to 1?

In fact, if we tried, we will know 1 is not right. Because we need every container can keep its proportion when it grows.

Demo：http://jsbin.com/tisaluy/6/edit?html,css,output

If we set the flex-grow of flex item to 1, the container proportion is different from the images then. In this demo I set the images height to the container’s height for better observation. 

Through some simple calculations, we will find that the width of each image(which is also the container) in the horizontal direction is just the percentage that its with from the total with of all images in this row.

In the case that it were not growed, the container width of each picture would been prorated. The remaining space of each line, we also want it to distribute in the current proportion of the width of the container, so the value of each containers flex-grow, is just its width, but not with the `px` unit.

The final code are shwon as:

```html
<div>flex，wrap
  //Acturally, flex-grow is allocated by proportion, so the `*200` in the second formula is not required. So we just need to change the previous “*200” as we need.
  <div style="width:{{img.width*200/img.height}}px;flex-grow:{{img.width*200/img.height}}" ng-repeat="img in imgs">
    <div style="padding-bottom:{{img.height/img.width*100}}%"></div>
  </div>
</div>
```

As a result, the current line will be filled with the container, and the same aspect ratio will be kept as the internal image which will put into the it:

Demo: http://jsbin.com/tisaluy/8/edit?html,css,output

As for how to deal with the last line, you can use a great element of flex-grow to fill the remaining space just as I described above.

After rendering the layout, you can feel free to resize and zoom, even without JS, the layout will not be in disorder.

Here, we finally achieve an image layout similar to Google Photos / 500px .

Summarize of the principles of this solution:

1. When padding and margin are percentage, the **width** of its container are used as a reference.
2. Use the flex-grow to allocate extra horizontal space according to the proportion of the image width  
3. Use the child elements with a fixed aspect ratio to stretch its parent and keep it to the same the aspect ratio.

Advantages of this layout:

* No special algorithm are required to compute the information like height or width of each image before rendering 
* Do not need to recalculate the layout during resize or zoom
* CSS solution is easy to integrate with any framework

Finally, we will talk about some of the shortcomings of this solution:

* Obviously, we can only define a minimum height(e.g. 200px in the above all demos) of the images in each row, and then wait for it to grow, and there is no way to specify the height of each row's upper limit. although we can set up the max-height of them , the images of the container affected by max-height will not display completely. In fact, as long as the ratio of the picture are in a normal range, any single row could hardly have too large height.
* the highest aspect ratio we allowed can be used when we encounters any image that exceed a certain number or aspect ratio. for example, met with a 1:5 images, we only use a 1:3 area to show it or you can adjust the order of the images, but this requires JS calculation. The search result page of “500px” seemingly work in this way. if you try to adjust the window width  to a small value, you will find the images behind the first picture cannot display complete, because if it display completely, a single image will take up too large screen space, so it limits the height.
* In addition, that are not likely to be ideal, because even if the images in last line nearly fill it, it can not because of placeholders, (you can adjust the width of the window to observe the complete demo above). But this situation can be avoided if JS calculation are used : when the last line almost filled, just fill it(I have came up a solution for this situation after two months when I wrote this post, I will probably write another post to address it).
* When it comes to the standard of image layout, the closer every image dispay, the higher score it should be got. In theory ,if you use JS to compute the layout, the following optimal algorithm can be used: if more taller images are in one row, less images should be put in the row, then more space can leave for the taller images in this row, so that the display areas of the taller and wider images will be closer to each other. But if you do not change the order of the images it would not be able to do this kind of optimization in CSS. And if you do not change the order of the images, even if the JS algorithm are optimized, the display areas of the wider images may be too large in the row which have more taller images, it also not perfect because of imbalance.

### The graceful degradation

Since flexbox is not supported in IE9 and below, graceful degradation of this layout is necessary. It should not be too bad to display these pictures in square on a browser which lack the flexbox support. Then float or inline-block can be used to break the line which it is not detailed here.

Finally, float the layout by day will create a layout like Google Photos which images can display in one row if images of consecutive days are few. Again, I came up with another method to address the Google Photos like layout which I'll write it in the next post.

##### That's all, folks! If you have any thoughts or questions, feel free to leave a comment!
