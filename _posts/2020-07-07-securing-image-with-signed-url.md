---
title: Securing Image With Signed URL
comments: true
categories:
- architecture
layout: post
date: '2020-07-07 10:00:00 +0700'
---

![Securing Image With Signed URL](/assets/Hero_ccims_18366839824_0.jpg)

A few days ago, I talked with my friend who works in startup which provide law related service. His company applied [KYC](https://www.thalesgroup.com/en/markets/digital-identity-and-security/banking-payment/issuance/id-verification/know-your-customer) to get users verified with it's identity card, one of his concern is that the way he retrieve the image of user's identity. Anyone with the link of the image, could open it anytime.

If the API to retrieve those image url is leaked, or the image link inadvertently shared to the internet, users will be harmed. To prevent that happen, we need to securing our API both to retrieve the URL and rendering the image.

After surfing the internet, i've found a terms of signed url. So, it's basically a URL with a signature and an expire time to accessing the resource.  This is example from GCS signed URL.

Without signature:

```
https://storage.googleapis.com/example-bucket/cat.jpeg
```

With signature:

```
https://storage.googleapis.com/example-bucket/cat.jpeg?X-Goog-Algorithm=
GOOG4-RSA-SHA256&X-Goog-Credential=example%40example-project.iam.gserviceaccount
.com%2F20181026%2Fus-central-1%2Fstorage%2Fgoog4_request&X-Goog-Date=20181026T18
1309Z&X-Goog-Expires=900&X-Goog-SignedHeaders=host&X-Goog-Signature=247a2aa45f16
9edf4d187d54e7cc46e4731b1e6273242c4f4c39a1d2507a0e58706e25e3a85a7dbb891d62afa849
6def8e260c1db863d9ace85ff0a184b894b117fe46d1225c82f2aa19efd52cf21d3e2022b3b868dc
c1aca2741951ed5bf3bb25a34f5e9316a2841e8ff4c530b22ceaa1c5ce09c7cbb5732631510c2058
0e61723f5594de3aea497f195456a2ff2bdd0d13bad47289d8611b6f9cfeef0c46c91a455b94e90a
66924f722292d21e24d31dcfb38ce0c0f353ffa5a9756fc2a9f2b40bc2113206a81e324fc4fd6823
a29163fa845c8ae7eca1fcf6e5bb48b3200983c56c5ca81fffb151cca7402beddfc4a76b13344703
2ea7abedc098d2eb14a7
```

Common architectural pattern which used storage bucket as it's component maybe looks like this.

![Image 1](/assets/Securing%20Image%20With%20Signed%20URL-1.png)

And here is the illustration about how url signature works.

Retrieve user data

![Retrieve user data](/assets/securing-image-with-signed-url-3-Get%20Image%20URL.png)

Get image

![Get image](/assets/securing-image-with-signed-url-3-Retrieve%20Image.png)

So the main key of the implementation is the algorithm and secret key that not exposed to a client, both of this must be same when retrieving and getting the image. Also, this is already supported by common used frameworks right now.

- [Laravel](https://laravel.com/docs/7.x/urls#signed-urls)
- [Djagon](https://docs.djangoproject.com/en/3.0/topics/signing/)

Yes, i know. It maybe not fully protecting the data being access by a random stranger online **if they had the link**. But it does have some protection because it has to be accessed within certain time and after that, the data will not remain available.

So, the conclusion is signed URL is the best way to protecting your image / files (AFAIK). You may choose to implement it or not at all,  depends on your cases. But the real software engineer is the one that place the right thing in the right place. Just like uncle Bob said, in it's speech about the scribe's oath (seriously you should check it out). Never produce harmfull code.


---

Example:

- [Laravel](https://github.com/rudestewing/api-mypost#penjelasan-singkat)
- [Golang](https://github.com/insomnius/example-signed-url)
- Comment out your example in the comment section


References:

[The scribe's oath by uncle Bob. Seriously, watch this!!](https://www.youtube.com/watch?v=Tng6Fox8EfI&t=536s)

[AWS Cloudfront private content signed urls](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html)

[GCS Storage Access Control with Signed URLS](https://cloud.google.com/storage/docs/access-control/signed-urls)
