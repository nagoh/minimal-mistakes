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

## Display Problem  

The above worked for me really well, and seemingly quite reliably. That was until I unplugged my monitor.

### Setup
Currently at work I use a 1920x1080 monitor as my primary display, and my laptop has a larger res (3200x1800). The laptop is almost impossible to use at this resolution unless you're using scaling to make all the things bigger. This is what i've got setup below:

![Display Settigns]({{ "/assets/images/display-settings.png" | relative_url }})

## Impact

Once I was running the primary screen with 200% scaling, things just stopped working as far as the webdriver logic goes. I was finding the cropped image being nowhere near the element. 

## Solution

What I did was add the scaling value into the webdriver code. In my case of 200%, I had to double the `X` & `Y` and `Size` values. Example below

{% highlight cs %}
public string GetCaptchaImageFromPage()
{
    // Take a screenshot
    var screenshot = _webDriver.GetScreenshot();

    // Convert the screenshot stream into an SixLabours.ImageSharp.Image
    var screenshotImage = Image.Load(screenshot.AsByteArray);

    // find the element on the page, so we can use it's location & size
    var element = _webDriver.FindElementById("imageToCrop");

    // get the resolution config value
    int res = int.Parse(_configuration["screen:resolutionMagnification"]);

     // create a rectangle so we can use this in a Crop() operation, with the scaling value
    var rectangleToCrop = new Rectangle(
                element.Location.X * res, 
                element.Location.Y * res, 
                element.Size.Width * res, 
                element.Size.Height * res);

    // Crop and return a Base64 string
    using (var childImage = screenshotImage.Clone(x => x.Crop(rectangleToCrop)))
    {
        return childImage.ToBase64String(ImageFormats.Jpeg);
    }
}
{% endhighlight %}

Hope that helps someone