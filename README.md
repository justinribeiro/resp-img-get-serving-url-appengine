Serving Responsive Images via Google Cloud Storage and Images Python API
==================================

Google Cloud storage can do a lot of interesting things by itself. If you combine it with [Images Python API](https://developers.google.com/appengine/docs/python/images/), you can use the =sXXX param to get properly scaled images that you can use for various breakpoints via picture or src-set.

### Prerequisites

1. You can either create a new project in the [Google Developers Console](https://console.developers.google.com) and use that or you can integrate the very basic route into an existing project. Up to you.
2. You'll need the [Google Cloud SDK](https://developers.google.com/cloud/sdk/) to be able to use gsutil.

### Setup

1. Clone this repo
```
git clone https://github.com/justinribeiro/resp-img-get-serving-url-appengine.git
```
2. Install the requirements so that our project will run (Flask, et cetera)
```
pip install -r requirements.txt -t lib
```
3. Change the application project id in app.yaml.
4. Deploy to App Engine (either via appcfg or via Git hook)

### How it works

1. Let's presume that we don't use our default bucket for our recently created project, but rather need to create a new one:

```
gsutil mb gs://myawesomebucket-pub/
```

2. Now let's set the base acl for the bucket:
```
gsutil acl set public-read gs://myawesomebucket-pub/
```

3. Let's copy a file to the bucket:
```
gsutil cp rock-bench-knights-ferry.jpg gs://myawesomebucket-pub/
```

4. Oh no, our acl on our upload didn't listen, let's set the acl on the file.
```
gsutil acl set public-read gs://myawesomebucket-pub/rock-bench-knights-ferry.jpg
```

5. Awesome! Our file is available via ```https://storage.googleapis.com/{bucket-name}/{file-name}``` but that's not what we want.

6. Now let's getting a serving url:

```
curl --data "bucket=myawesomebucket-pub&image=rock-bench-knights-ferry.jpg" SOMETHING.appspot.com/serveurl
```

Which will return a url that looks something like:

```
https://lh5.ggpht.com/rCGu26RVMiLkEepYZhhfmxMxsKrb29wUFGfqirbErbNvmLqVlr7mFvXILGQrSZ_u53D4OpMSh_wN3lUoh224RhWWFJlFQA
```

7. Now we can drop this into our ```<picture>``` tag as by using the =sXXX argument on the url to give us happy correct sized images.

```
<picture>
	<source srcset="https://lh5.ggpht.com/rCGu26RVMiLkEepYZhhfmxMxsKrb29wUFGfqirbErbNvmLqVlr7mFvXILGQrSZ_u53D4OpMSh_wN3lUoh224RhWWFJlFQA=s1000" media="(min-width: 1000px)">
	<source srcset="https://lh5.ggpht.com/rCGu26RVMiLkEepYZhhfmxMxsKrb29wUFGfqirbErbNvmLqVlr7mFvXILGQrSZ_u53D4OpMSh_wN3lUoh224RhWWFJlFQA=s800" media="(min-width: 800px)">
	<source srcset="https://lh5.ggpht.com/rCGu26RVMiLkEepYZhhfmxMxsKrb29wUFGfqirbErbNvmLqVlr7mFvXILGQrSZ_u53D4OpMSh_wN3lUoh224RhWWFJlFQA=s400">
	<img srcset="https://lh5.ggpht.com/rCGu26RVMiLkEepYZhhfmxMxsKrb29wUFGfqirbErbNvmLqVlr7mFvXILGQrSZ_u53D4OpMSh_wN3lUoh224RhWWFJlFQA=s400" alt="A peaceful day down at Knights Ferry.">
</picture>
```
You can see this in action via the following JSFiddle: [http://jsfiddle.net/justinribeiro/kTVHd/](http://jsfiddle.net/justinribeiro/kTVHd/). The fiddle uses the very awesome [Picturefill image polyfill](https://github.com/scottjehl/picturefill).

### I need CORS Justin!?!?!?!?!

Google Cloud Storage does have support for cross-origin resource sharing [CORS](https://developers.google.com/storage/docs/cross-origin):
```
gsutil cors set my-cors.json gs://myawesomebucket-pub
```

where the json-file looks like:
```
[
  {
    "origin": ["https://my.domain.somewhere"],
    "responseHeader": ["Content-Type"],
    "method": ["GET"],
    "maxAgeSeconds": 3600
  }
]
```


### Gotcha's and things

1. Only one app can "own" the image. As stated in the [documentation](https://developers.google.com/appengine/docs/python/images/functions) for get_serving_url:

> If you serve images from Google Cloud Storage, you cannot serve an image from two separate apps. Only the first app that calls get_serving_url on the image can get the URL to serve it because that app has obtained ownership of the image.

2. Can't scale up above 1600 pixels. As a matter of fact, the Image service won't scale an image beyond the uploads intial size (don't expect the service to scale 32px image to 1600px).

3. Probably other things that I'm forgetting at the moment.
