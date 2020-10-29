---
title: "大文件上传实现"
subtitle: ""
date: 2020-10-30T05:11:59+08:00
lastmod: 2020-10-30T05:11:59+08:00
draft: false
author: "steinw"
authorLink: "steinw.cc"
description: ""

tags: [
  "web"
]
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->
> 不少面试会面这个问题，于是这边自己整理的同时顺便实现了一下。

## 前言

demo在 <https://github.com/sjisntsuperman/some-demo> 里面， 有兴趣的可以自己clone下来使用。其中部分代码引用摘录于网上的文章，这里是自己再整理。所以可能会有些出入。

## 客户单实现

首先实现 utils 工具库。这里要用到的有，请求库 以及 请求并发限制的方法。

分别来实现它。因为这边可能会遇到请求阻塞的情况，所以这边写了个并发请求限制的函数，来预防阻塞的情况发生。

### 并发请求限制

这也是头条经常面的面试题，顺便回顾一下。

```typescript
/**
 * @name 并发请求限制
 * @param maxLimit
 * @param requestList
 */

export const asyncPool: Function = (maxLimit: number, requestList: Array<object>, iteratorFn:Function) => {
  const executing: Promise<void>[] = []
  const res: Promise<void>[] = []
  let i = 0

  const equeue = (): Promise<any> => {
    if (i === requestList.length) {
      return Promise.resolve(res)
    }

    const requestOptions = requestList[i++]
    const request: Promise<any> = Promise.resolve().then(()=>iteratorFn(requestOptions));
    res.push(request)
    const block: Promise<any> = request
      .then(() => {
        return executing.splice(executing.indexOf(request), 1)
      })
    executing.push(block)

    let r = Promise.resolve()

    if (executing.length >= maxLimit) {
      r = Promise.race(executing)
    }

    return r.then(() => equeue())
  }

  return equeue().then(() => Promise.all(res))
}

```

### 请求

```typescript
type requestOptions = {
  url?: string
  method?: string
  data?: FormData|string
  headers?: CustomT
  onProgress?: {
    (this: XMLHttpRequest, ev: ProgressEvent) : any | null
  },
  requestList?: Array<XMLHttpRequest>
}

interface fetchMethod {
  (propName: requestOptions): Promise<any>
}

const fetch :fetchMethod= ({
  url='',
  method = 'post',
  data='',
  headers = {},
  onProgress = (e: ProgressEvent<EventTarget>) => e,
  requestList = []
}: requestOptions) => {
  return new Promise((resolve) => {
    const xhr: XMLHttpRequest = new XMLHttpRequest()
    xhr.upload.onprogress = onProgress
    xhr.open(method, url)
    Object.keys(headers).forEach(key => xhr.setRequestHeader(key, headers[key]))
    xhr.send(data)
    xhr.onload = (e: any) => {
      resolve({
        data: e?.target?.response,
      })
    }
    requestList.push(xhr)
  })
}
```

接下来是，组件部分的实现。

### 组件部分的实现

通过 worker 计算出 hash 之后， 需要分包。

也就是将文件按照 固定的大小 进行分开，以流的形式，切开几个Buffer。

### 计算文件hash

这里因为不影响主线程的正常运作，我们用Worker线程进行计算hash。减少主线程的压力。

```typescript
// 这是在vue文件中
creatHash(chunks: Array<object>) {
    return new Promise(resolve => {
      this.container.worker = new Worker('/hash.js')
      this.container.worker.postMessage({chunks})
      this.container.worker.onmessage = (e: any) => {
        const {percentage, hash} = e.data
        this.hashPercentage = percentage
        if (hash) {
          resolve(hash)
        }
      }
    })
  }
```

```js
// worker 脚本
self.importScripts("/spark-md5.min.js"); // 导入脚本

// 生成文件 hash
self.onmessage = e => {
  const { chunks } = e.data;
  const spark = new self.SparkMD5.ArrayBuffer();
  let percentage = 0;
  let count = 0;
  const loadNext = index => {
    const reader = new FileReader();
    reader.readAsArrayBuffer(chunks[index].file);
    reader.onload = e => {
      count++;
      spark.append(e.target.result);
      if (count === chunks.length) {
        self.postMessage({
          percentage: 100,
          hash: spark.end()
        });
        self.close();
      } else {
        percentage += 100 / chunks.length;
        console.log(percentage)
        self.postMessage({
          percentage
        });
        loadNext(count);
      }
    };
  };
  loadNext(0);
};
```

### 分割chunk

arrayBuffer可以使用`Array.prototype.slice`来切割

```js
 createChunks(file:Blob) {
    const chunks = []
    let cur = 0
    while (cur < file.size) {
      chunks.push({file: file.slice(cur, cur + SIZE)})
      cur += SIZE
    }
    return chunks
  }
```

分割之后还有上传逻辑。

### 上传切片

```typescript
  async upload() {
    console.log(this.container)
    if (!this.container.file) return
    const chunks = await this.createChunks(this.container.file)
    this.container.hash = await this.creatHash(chunks)

    // 先检查有没已上传的文件列表
    await this.checkChunks()

    this.data = chunks.map(({file}, index) => ({
      filename: this.container.file.name,
      fileHash: this.container.hash,
      index,
      hash: this.container.hash + '-' + index,
      chunk: file,
      size: file.size,
      percentage: this.uploadedList.includes(index) ? 100 : 0,
    }))
    this.uploadChunks()
  }

  async uploadChunks() {
    // 避免重新传已有的文件
    const requestList: CustomTS = this.data
      .filter((flieInfo: CustomTS) => !this.uploadedList.includes(flieInfo.hash))
      .map((fileInfo: CustomTS) => {
        const {index} = fileInfo
        const form: FormData = this.createFormData(fileInfo, 'chunk', 'hash', 'fileHash', 'filename')
        return {
          url: prefix + 'upload',
          data: form,
          onProgress: this.progressHandler(this.data[index]),
          requestList: this.requestList,
        }
      })
    this.status = STATUS.PENDING
    this.msg = 'file is been uploading'
    //请求并发限制
    await asyncPool(4, requestList, fetch)
    this.mergeChunk().then(() => {
      this.status = STATUS.FULFILLED
      this.msg = 'file has been uploaded'
    })
  }
```

### 合并切片

```typescript
mergeChunk() {
    return fetch({
      url: 'http://localhost:4000/merge',
      headers: {
        'content-type': 'application/json',
      },
      data: JSON.stringify({
        filename: this.container.file.name,
        size: SIZE,
        fileHash: this.container.hash,
      }),
    })
  }
```

### 进度条逻辑

```js
// 监听 xhr 的 onProgress
function progressHandler(fileInfo: CustomTS) {
    return (e: ProgressEvent) => {
      fileInfo.progress = Math.floor(e.loaded / e.total) / 100
    }
  }
```

## 服务端实现

服务端的实现相对简单点。这里需要用到，一些cors 的方法。

因为客户端用了复杂请求，所以这里要额外处理一下遇见请求。至于为何客户端会发预检请求，是因为 浏览器检查是否跨域使用的。

```typescript
const http = require('http')
// const
const server = http.createServer()

server.on("request", async (req, res) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Headers", "*");
  if (req.method === "OPTIONS") {
    res.status = 200;
    res.end();
    return;
  }

  if(req.url === '/'){
    return res.end("Hello UploadLoader");
  }

  if (req.url === "/check") {
    await CheckChunkController(req, res);
    return;
  }

  if (req.url === "/merge") {
    await MergeController(req, res);
    return;
  }

  if (req.url === "/upload") {
    await UploadController(req, res);
  }
});
```

### uploadHandle

这个句柄具体要处理的是把接受到的buffer写入到本地文件中。

```typescript
export const UploadController: Controller = async (req, res) => {
const multipart = new multiparty.Form({});

multipart.parse(req, async (err, fields, files) => {
    if (err) {
        logger.error(err);
        res.statusCode = 500;
        res.end('process file chunk failed');
        return;
    }
    // logger.info(JSON.stringify(fields))
    const [ chunk ] = files.chunk;
    const [ hash ] = fields.hash;
    const [ fileHash ] = fields.fileHash;
    const [ filename ] = fields.filename;
    const filePath = path.resolve(UPLOAD_DIR, `${fileHash}${extractExt(filename)}`);
    const chunkDir = path.resolve(UPLOAD_DIR, fileHash);

    // 如果不存在文件夹则创建
    if (!fs.existsSync(UPLOAD_DIR)) {
        fs.mkdirSync(UPLOAD_DIR);
    }

    // 文件存在直接返回
    if (fs.existsSync(filePath)) {
        res.end('file exist');
        return;
    }

    // 切片目录不存在，创建切片目录
    if (!fs.existsSync(chunkDir)) {
        await fs.mkdirSync(chunkDir);
    }
    // fs-extra 专用方法，类似 fs.rename 并且跨平台
    // fs-extra 的 rename 方法 windows 平台会有权限问题
    // https://github.com/meteor/meteor/issues/7852#issuecomment-255767835
    await fs.move(chunk.path, path.resolve(chunkDir, hash));
    res.end('received file chunk');
});
};
```

### mergeHandle

这个句柄主要是将buffer合并。

```typescript
export const MergeController: Controller = async (req, res) => {
const promisedata: any = await bodyParser(req);
const { size, filename, fileHash } = promisedata;
const chunkDir = path.join(UPLOAD_DIR, `${fileHash}`);
const files = fs.readdirSync(chunkDir);
const ext = extractExt(filename);
// output dir
const pathname = path.join(UPLOAD_DIR, `${fileHash}${ext}`);
// 避免顺序混乱
files.sort((prev, cur) => Number(prev.split('-')[1]) - Number(cur.split('-')[1]));
await Promise.all(
    files.map((chunkpath: string, i: number) =>
        pipeStream(
            path.join(chunkDir, chunkpath),
            fs.createWriteStream(pathname, {
                start: size * i
            })
        )
    )
);
fs.rmdirSync(chunkDir);
res.end(
    JSON.stringify({
        retcode: 0,
        message: 'merge success'
    })
);
};
```

### checkHandle

这个句柄主要检查本地有没已上传的切片。

```typescript
export const CheckChunkController: Controller = async (req, res) => {
const { filename, hash, filehash } = await bodyParser(req);
const ext = extractExt(filename);
const chunkpath = path.join(UPLOAD_DIR, `${hash}${ext}`);
const existChunk = verifyChunk(filehash, filename);
if (existChunk) {
    res.end('file is already existed');
    return;
}
const existChunks = await fs.existsSync(chunkpath);
if (!existChunks) {
    res.end(
        JSON.stringify({
            uploadedList: []
        })
    );
} else {
    const uploadedList = await fs.readdirSync(chunkpath);
    res.end(
        JSON.stringify({
            uploadedList: uploadedList
        })
    );
}
};

```
