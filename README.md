# qtcurl
Run `curl_easy_*` transfers asynchronously just inside your **Qt** main thread in the right way.

Not very much of code needed for that actually. But if you too lazy to learn how to deal with *curl_multi* in event-based systems, you'll probably be happy with these sources. 

Now there's just a pretty straight wrapper around *curl_easy* functions with some C++/Qt sugar for HTTP headers and common callbacks. More things may be added later.



##Usage:
###1. Set up linking and includes for *libcurl*
Be strong and do it yourself. There should be plenty of docs for this.
###2. Include qtcurl.pri into your *.pro*
```qmake
include (qtcurl/qtcurl.pri)
```
###3. Write your code
Include header, please:
```c++
#include "CurlEasy.h"
```

Create the transfer object and set it up:
```c++
	CurlEasy *curl = new CurlEasy;
	curl->set(CURLOPT_URL, "https://www.google.com");
	curl->set(CURLOPT_FOLLOWLOCATION, long(1)); // Tells libcurl to follow HTTP 3xx redirects
```

You also may want to override some curl callbacks:
```c++
  curl->setWriteFunction([](char *data, size_t size)->size_t {
		qDebug() << "Data from google.com: " << QByteArray(data, static_cast<int>(size));
		return size;
	});
 ```

And set some HTTP headers:
```c++
  curl->setHttpHeader("User-Agent", "My poor little application");
```

Oh, the signals are there, of course!
```
	QObject::connect(curl, &CurlEasy::done, [&application](CURLcode result) {
		qDebug() << "Transfer for google.com is done with CURL code: " << result;
	});
  
	QObject::connect(curl, &CurlEasy::done, transfer, &CurlEasy::deleteLater);
```

Now it's time to activate the transfer and pass it to all those event-looping things:
```
	curl->perform();
  
  // Do your stuff while the request is processing.
```

That's it. You may look into the sources for more details =)
