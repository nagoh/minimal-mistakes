---
classes: wide
category: software
tags: [selenium, .net, webdriver, windows]
---

## Problem 

Recently I had to extract a captcha image from a page. The image was a randomly generated Captcha image, and generated a new image for every request to the image url. 

The image had a static query string parameter, i.e:
```html
<a src="/img.jpg?0.16065923657102799" />
```
However, requesting this each time returned a different Captcha  image. 

## Webdriver 

I came across [this](https://stackoverflow.com/questions/13832322) solution when trying to extract the exact image that was bring rendered on the page. It seemed to work pretty well, and I ended up with an implementation using [ImageSharp](https://github.com/SixLabors/ImageSharp). This is what I ended up with:

{% highlight cs %}
public string GetCaptchaImageFromPage()
{
    // Take a screenshot
    var screenshot = _webDriver.GetScreenshot();

    // Convert the screenshot stream into an SixLabours.ImageSharp.Image
    var screenshotImage = Image.Load(screenshot.AsByteArray);

    //find the element on the page, so we can use it's location & size
    var element = _webDriver.FindElementById("imageToCrop");

    // create a rectangle so we can use this in a Crop() operation
    var rectangleToCrop = new Rectangle(element.Location.X, element.Location.Y, element.Size.Width, element.Size.Height);

    // Crop and return a Base64 string
    using (var childImage = screenshotImage.Clone(x => x.Crop(rectangleToCrop)))
    {
        return childImage.ToBase64String(ImageFormats.Jpeg);
    }
}
{% endhighlight %}