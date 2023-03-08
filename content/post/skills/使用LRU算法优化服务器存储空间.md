---
title: "使用LRU算法优化服务器存储空间"
date: 2023-01-15T10:30:15+08:00
draft: true
tags: ['lru', '算法']
---

为了获取生产环境报错对应的源码位置，之前将打包之后的 **map 文件**上传到 node 服务，日积月累之下，已经占用了好几个 g 的磁盘空间

就想到用 `LRU 算法` 的思想去对这些文件进行管理

## LRU

`LRU` 英文全称是 Least Recently Used，最不经常使用，是一种内存管理算法。可以参考 `leetcode` 的描述 [LRU缓存机制](https://leetcode.cn/problems/lru-cache/)

核心思想是 **“如果数据最近被访问过,那么将来被访问的几率也更高”**。所以在容量有限的情况下，淘汰最近最少使用的缓存。即近期内最不会被访问的数据进行淘汰

可以使用一个队列来缓存数据

如果有**新数据插入**，就移出队列中最后一个缓存

![新增数据](https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/5dc1816247c0fc068c96773a1ccd6bc3.jpg)

如果有**数据访问**，要将访问到的数据移到尾部，重新进行排序

![访问数据](https://findmio.oss-cn-hangzhou.aliyuncs.com/blog/fec5b1d3878cda6ccb70ced5a6d5d433.jpg)



## 实现

在 `javascript` 中，可以使用 `Map` 或者 `Set` 来模拟一个队列



 

```typescript
import path from "path";
import fse from "fs-extra";

/** 上传 sourcemap 存放地址 */
export const sourcemapFilePath = path.join(__dirname, "../../../../uploads");

// 使用 LRU 算法去控制 sourcemap 文件
export class SourcemapCache {
    private path = path.join(__dirname, "cache.json");
    private length: number; // 最大缓存数量
    private cache: Set<string>;

    constructor(length = 10) {
        this.length = length;
        this.cache = new Set(this.readCache() || []);
    }

    private readCache = () => {
        try {
            return fse.readJSONSync(this.path);
        } catch (error) {
            return;
        }
    };

    private saveCache(date: string[]) {
        fse.ensureFile(this.path).then(() => {
            fse.writeJSONSync(this.path, date);
        });
    }

    private delete(key: string) {
        fse.remove(path.resolve(sourcemapFilePath, key));
    }

    // 访问 缓存中的某一项
    access(key: string) {
        if (this.cache.has(key)) {
            this.cache.delete(key);
            this.cache.add(key);
            this.saveCache([...this.cache.keys()]);
        }
    }

    // 数据插入，重新排序
    put(key: string) {
        const keys = [...this.cache.keys()];
        // 已存在队列，则移到尾部
        if (this.cache.has(key)) {
            this.access(key);
        } else {
            if (keys.length >= this.length) {
                const oldestKey = keys[0];
                this.cache.delete(oldestKey);
                this.delete(oldestKey);
            }
            this.cache.add(key);
            this.saveCache([...this.cache.keys()]);
        }
    }
}
```
