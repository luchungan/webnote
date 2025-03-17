# utils工具函数

```typescript
// 判断当前环境
// 1.在构建工具中配置： 如果你使用的是 Webpack，可以在 Webpack 配置文件中通过 DefinePlugin 插件来设置 NODE_ENV： 
 new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
    }),
//2.在运行脚本中配置： 在 package.json 的 scripts 部分，可以通过命令行参数设置 NODE_ENV：
//"NODE_ENV=development pnpm run dev"

// 3.使用环境变量文件： 你可以使用 .env 文件来配置环境变量，并使用 dotenv 库在项目中加载这些变量：

// # filepath: .env
// NODE_ENV=development

const isDev = process.env.NODE_ENV === 'development' || process.env.NODE_ENV === 'test';
```

```typescript 
// 判断两个deps是否完全相等 
function depsAreSame(oldDeps: DependencyList, deps: DependencyList): boolean {
  if (oldDeps === deps) return true;
  for (let i = 0; i < oldDeps.length; i++) {
    if (!Object.is(oldDeps[i], deps[i])) return false;
  }
  return true;
}
/// Object.is 类似于 === ,区别 NaN 与 NaN 被认为是相等的。
      //+0 与 -0 被认为是不相等的。
     /// ===正好相反
```

```typescript
// 限制单位时间内 fn执行一次
function limit(fn: any, timespan: number) {
  let pending = false;
  return (...args: any[]) => {
    if (pending) return;
    pending = true;
    fn(...args);
    setTimeout(() => {
      pending = false;
    }, timespan);
  };
}
```

```typescript 缓存相关
const cache = new Map<CachedKey, RecordData>();

const setCache = (key: CachedKey, cacheTime: number, cachedData: CachedData) => {
  const currentCache = cache.get(key);
  if (currentCache?.timer) {
    clearTimeout(currentCache.timer);
  }

  let timer: Timer | undefined = undefined;

  if (cacheTime > -1) { // 设置过期时间，过期自动清除
    // if cache out, clear it
    timer = setTimeout(() => {
      cache.delete(key);
    }, cacheTime);
  }

  cache.set(key, {
    ...cachedData,
    timer,
  });
};

const getCache = (key: CachedKey) => { // 获取cache
  return cache.get(key);
};

const clearCache = (key?: string | string[]) => { // 清除cache
  if (key) {
    const cacheKeys = Array.isArray(key) ? key : [key];
    cacheKeys.forEach((cacheKey) => cache.delete(cacheKey));
  } else {
    cache.clear();
  }
};
```
