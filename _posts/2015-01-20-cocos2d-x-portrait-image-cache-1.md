---
layout: post
title: Cocos2d-x下载头像（一)
description: "Cocos2d-x中下载网络图片并缓存."
categories: Cocos2d-x
---

> Cocos2d-x中下载网络图片并缓存.

---

当前在游戏开发中，无论大型游戏还是休闲小游戏，都喜欢加入好友系统，那就涉及到获取好友头像的需求，那在Cocos2d-x中如果获取网络图片呢？

先封装一个下载图片的工具类，顺便做一下缓存吧，要不每次都下载多耗流量啊——

```cpp
std::string GImageUtils::downloadImageWithUrl(const std::string &fullUrl, const ccHttpRequestCallback & callback)
{
	// - - - - - - - - - - 图片缓存 - - - - - - - - - -
    std::string filePath = __String::createWithFormat("%s", FileUtils::getInstance()->getWritablePath().c_str())->getCString();

    std::vector<std::string> splitStr = GStringUtils::split(fullUrl.c_str(), "http://");
    if (splitStr.size() > 1) {
        filePath += GStringUtils::replace_all_distinct(splitStr.at(1), "/", "-") ;

        FILE* fp = fopen(filePath.c_str(),"rb");
        if (fp)
        {
            CCLOG("has file! filePath: %s", filePath.c_str());
            return filePath;// 如果有缓存图片则直接返回路径
        }
    }

    // - - - - - - - - - - 下载图片 - - - - - - - - - -
    auto request = new HttpRequest();
    // required fields
    request->setRequestType(HttpRequest::Type::GET);
    request->setUrl(fullUrl.c_str());
    request->setResponseCallback(callback);

    // optional fields
    request->setTag("_Http_GetImage_");
    HttpClient::getInstance()->send(request);
    request->release();
    return kImageNoCacheTag;// 没有缓存返回Tag区分情况处理
}

std::string GImageUtils::onGettedImage(HttpClient *sender, HttpResponse *response)
{
    if (!response) {
        return kImageDownloadErrorTag;
    }

    if (0 != strlen(response->getHttpRequest()->getTag())) {
        CCLOG("HttpConnect: - %s Completed. -", response->getHttpRequest()->getTag());
    }

    long statusCode = response->getResponseCode();
    char statusString[64] = {};
    sprintf(statusString, "HttpConnect: HTTP Status Code: %ld, tag = %s.", statusCode, response->getHttpRequest()->getTag());
    CCLOG("HttpConnect: response code: %ld", statusCode);

    if (!response->isSucceed()) {
        CCLOG("HttpConnect: response failed!!!...");
        CCLOG("HttpConnect: error buffer: %s.", response->getErrorBuffer());
        return kImageDownloadErrorTag;
    }

    // dump data
    std::vector<char> *buffer = response->getResponseData();

    if (strcmp(response->getHttpRequest()->getTag(), "_Http_GetImage_") != 0) {
        return kImageDownloadErrorTag;
    }

    // - - - - - - - - - - 下载成功，生成图片 - - - - - - - - - -
    auto img = new Image();
    img->initWithImageData((unsigned char*)buffer->data(), buffer->size());

    // - - - - - - - - - - 缓存图片 - - - - - - - - - -
    std::string filePath = __String::createWithFormat("%s", FileUtils::getInstance()->getWritablePath().c_str())->getCString();

    // 保存到本地文件，把图片网址作为文件名
    std::string bufffff(buffer->begin(),buffer->end());
    std::vector<std::string> splitStr = GStringUtils::split(response->getHttpRequest()->getUrl(), "http://");
    if (splitStr.size() > 1) {
        filePath += GStringUtils::replace_all_distinct(splitStr.at(1), "/", "-") ;
    }

    CCLOG("filePath: %s", filePath.c_str());
    FILE *fp = fopen(filePath.c_str(), "wb+");
    fwrite(bufffff.c_str(), 1, buffer->size(), fp);
    fclose(fp);

    img->release();

    return filePath;// 返回图片路径
}
```
