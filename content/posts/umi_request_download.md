---
title: "umi request 下载数据流"
date: 2022-04-29T17:09:30+08:00
draft: false
---

## 前言

今天接了个需求，用文件名去请求，然后后端返回数据回来，前端下载。以前都是 location 直接跳转下载，第一次用请求去下，踩了一点点坑。

## 问题

先是网上搜了一下，大部分人的接口返回后，可以直接 `res => res.blob()` 拿到 blob 数据，但是我这边后端返回不能直接这样子拿，返回的是一个 `ReadableStream` 的 body ，所以到手需要读取一下。

如果用 fetch 的话可以直接 `res => res.blob()` ，但是奇怪的是内容是空，状态码是 304 ，不知道后端那边是怎么处理的。

## 读取

主要参考了 MDN 的这篇文章 [使用可读文件流](https://developer.mozilla.org/zh-CN/docs/Web/API/Streams_API/Using_readable_streams) 。可以自己封装一下读取的方法

```ts
export async function getReadableStreamInfo(
  reader: ReadableStreamDefaultReader
) {
  const stream = new ReadableStream({
    start(controller) {
      return pump();
      function pump(): any {
        return reader.read().then(({ done, value }) => {
          if (done) {
            controller.close();
            return;
          }
          controller.enqueue(value);
          return pump();
        });
      }
    },
  });
  const response = new Response(stream);
  return await response.blob();
}
```

使用

```ts
const res = await downloadFile(reportName);
// 这样就能拿到 blob 了
const blob = await getReadableStreamInfo(res?.body?.getReader());
```

## 下载

封装一下方法，方便调用，这样就可以下载了。

```ts
export function downloadFileLink(link: string, fileName: string) {
  const a = document.createElement("a");
  document.body.appendChild(a);
  a.href = link;
  a.target = "_blank";
  a.download = fileName;
  a.click();
  a.remove();
  window.URL.revokeObjectURL(link);
}
```

## 总结

总的来说，如果后端返回内容比较大众化，网上的方法应该都适用，但如果后端返回比较奇怪，可能就需要这样读取一下 stream 的信息才能下载了
