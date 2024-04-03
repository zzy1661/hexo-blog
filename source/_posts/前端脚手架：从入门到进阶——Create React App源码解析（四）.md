---
 title: "前端脚手架：从入门到进阶——Create React App源码解析（四）"
 date: 2021-11-15
 tags: [前端]
 categories: [前端笔记]
---

这是我参与11月更文挑战的第 15 天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095/ "https://juejin.cn/post/7023643374569816095/")

CRA webpack配置
-------------

本文编写的时候react-scripts版本时4.0.3，还没有升级到webpack5，不过其仓库中已经有了webpack5相关的配置，因此下面的解析是基于其最新的代码。

create-react-app的webpack配置基本上集中在react-scripts/config/webpack.config.js，development和production的不同环境配置都在这一个文件里进行判断，将它拆开后更好阅读些：

首先是声明一些环境变量，这些变量基本集中在config目录下的paths.js、modules.js、env.js文件中，这里先进行一个预设：项目的入口为`D:\cra-demo1\src\index.tsx`，后续入口相关的参数将替换为这个值，这样更方便理解

*   最外层：导出的是一个函数

```javascript
module.exports = function (webpackEnv) {

}

```

webpack配置文件，可以导出一个对象，或者多个对象，也可以导出一个函数，或者一个promise，

当导出一个函数时，可以传入两个参数，第一是环境对象environment，第二个是传给webpack的选项

### target

target: webpack中文文档已经过期了，如果项目中有browserslist配置，webpack将会用它 - 确定可用于生成运行时代码的 ES 功能 - 推断环境，可以不再配置output.environment

```vbnet
target: ['browserslist'],
```

### mode

mode设置为`'production'`，会将 DefinePlugin 中 process.env.NODE\_ENV 的值设置为 production。启用 FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 TerserPlugin。

```arduino
 mode: isEnvProduction ? 'production' : isEnvDevelopment && 'development',,
```

### bail

bail:在生产环境编译遇到错误直接抛出并终止

```makefile
bail: isEnvProduction,
```

### devtool

devtool: 生产环境使用`shouldUseSourceMap`控制是否需要source map。在实践中，生产环境打包可能需要生成source map方便监控或日志平台进行定位，但也可能不需要，因此这里设置了一个变量，由`process.env.GENERATE_SOURCEMAP`控制

```vbnet
devtool: isEnvProduction
    ? shouldUseSourceMap
    ? 'source-map'
    : false
    : isEnvDevelopment && 'cheap-module-source-map',
```

### entry

entry：入口文件，比如这里是'D:\\cra-demo1\\src\\index.tsx'

```makefile

entry: paths.appIndexJs,
```

### output

output:相关字段配置和解释如下

```javascript
 output: {
    // 输出目录，比如这里是'D:\\cra-demo1\\build'.
    path: paths.appBuild,
     // 开发环境输出代码里增加/* filename */注释
    pathinfo: isEnvDevelopment,
    // 主bundle
    filename: isEnvProduction
    ? 'static/js/[name].[contenthash:8].js'
    : isEnvDevelopment && 'static/js/bundle.js',
    // 代码分隔后的chunk 文件
    chunkFilename: isEnvProduction
    ? 'static/js/[name].[contenthash:8].chunk.js'
    : isEnvDevelopment && 'static/js/[name].chunk.js',
    assetModuleFilename: 'static/media/[name].[hash][ext]',
    // 这里是'/'
    publicPath: paths.publicUrlOrPath,
    // source map路径
    devtoolModuleFilenameTemplate: isEnvProduction
    ? info =>
        path
            .relative(paths.appSrc, info.absoluteResourcePath)
            .replace(/\\/g, '/')
    : isEnvDevelopment &&
        (info => path.resolve(info.absoluteResourcePath).replace(/\\/g, '/')),
},
```

### cache

cache: webpack5自带的缓存，加快构建速度

```arduino
cache: {
    // 将缓存保存在文件中
    type: 'filesystem',
    // 缓存版本，版本更新将使原缓存失效
    version: createEnvironmentHash(env.raw),
    // 缓存的目录：node_modules/.cache
    cacheDirectory: paths.appWebpackCache,
    // 当编译器空闲时将数据存储在一个文件中，用于所有缓存项
    store: 'pack',
    // 缓存的依赖，这里的变更将使缓存失效
    buildDependencies: {
        defaultWebpack: ['webpack/lib/'],
        config: [__filename],
        tsconfig: [paths.appTsConfig, paths.appJsConfig].filter(f =>
            fs.existsSync(f)
        ),
    },
},
```

### infrastructureLogging

infrastructureLogging:基础日志级别，这里不开启，cra有自己的日志

```css
infrastructureLogging: {
    level: 'none',
},
```

### optimization

*   optimization: 代码优化，比如压缩、分割等

```yaml
optimization: {
    // 生产环境进行压缩
    minimize: isEnvProduction,
    minimizer: [
        js和css压缩...
    ],
},
```

*   js压缩

```yaml
new TerserPlugin({
    terserOptions: {
    parse: {
        // 以es8语法解析
        ecma: 8,
    },
    compress: {
        ecma: 5,
        warnings: false,
        comparisons: false,
        inline: 2,
    },
    mangle: {
        // 解决Safari 10中的一个bug
        safari10: true,
    },
    // 是否保留classnames
    keep_classnames: isEnvProductionProfile,
    keep_fnames: isEnvProductionProfile,
    output: {
        ecma: 5,
        comments: false,
        ascii_only: true,
    },
    },
}),
```

*   css 压缩

```scss
    new CssMinimizerPlugin(),
```

### resolve

resolve:帮助找到模块的路径

*   modules:从哪里找模块

```css
resolve: {
    // 这个配置主要考虑了monorepo的场景
    modules: ['node_modules', paths.appNodeModules].concat(
        modules.additionalModulePaths || []
    ),
   ...
},
```

*   extensions: 可以省略的后缀名，包含了： \[ 'web.mjs', 'mjs', 'web.js', 'js', 'web.ts', 'ts', 'web.tsx', 'tsx', 'json', 'web.jsx', 'jsx', \];

```ini
 extensions: paths.moduleFileExtensions
        .map(ext => `.${ext}`)
        .filter(ext => useTypeScript || !ext.includes('ts')),
    
```

*   alias:modules.webpackAliases中只有`src`

```csharp
alias: {
        'react-native': 'react-native-web',
        // Allows for better profiling with ReactDevTools
        ...(isEnvProductionProfile && {
            'react-dom$': 'react-dom/profiling',
            'scheduler/tracing': 'scheduler/tracing-profiling',
        }),
        // 这里只设置了src
        ...(modules.webpackAliases || {}),
    },
    
```

*   plugins:

```arduino
plugins: [
    // 这个插件用来防止用户从src之外的地方导入文件
        new ModuleScopePlugin(paths.appSrc, [
            paths.appPackageJson,
            reactRefreshRuntimeEntry,
            reactRefreshWebpackPluginRuntimeEntry,
            babelRuntimeEntry,
            babelRuntimeEntryHelpers,
            babelRuntimeRegenerator,
        ]),
    ],
```

### module

module: 如何处理不同类型的模块

*   strictExportPresence

```arduino
module: {
    // 将缺失的导出作为error，而不是warning
    strictExportPresence: true,
    ...
   
},
```

*   rules: 这里用了`oneOf`api，遇到第一个匹配的就会终止，如果没有匹配的，就会执行最下面的

```yaml
 rules: [
        // 处理第三方库的source map
        shouldUseSourceMap && {
            enforce: 'pre',
            exclude: /@babel(?:\/|\\{1,2})runtime/,
            test: /\.(js|mjs|jsx|ts|tsx|css)$/,
            loader: require.resolve('source-map-loader'),
        },
        {
            // "oneOf" 遍历下面所有的loader，直到第一个符合的，如果没有找到，则使用最下面的'file loader'
            // webpack5取消了file-loader，因此这里加了个引号
            oneOf: [
                {
                    test: [/\.avif$/],
                    type: 'asset',
                    mimetype: 'image/avif',
                    // 这个是webpack5的配置，取消raw-loader、url-loader、file-loader
                    parser: {
                        dataUrlCondition: {
                            maxSize: imageInlineSizeLimit,
                        },
                    },
                },
                {
                    test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
                    type: 'asset',
                    parser: {
                    dataUrlCondition: {
                        maxSize: imageInlineSizeLimit,
                    },
                    },
                },
                {
                    test: /\.svg$/,
                    use: [
                        {
                            //可以将svg以组件的形式导入 import Star from './star.svg'
                            loader: require.resolve('@svgr/webpack'),
                            options: {
                                prettier: false,
                                svgo: false,
                                svgoConfig: {
                                    plugins: [{ removeViewBox: false }],
                                },
                                titleProp: true,
                                ref: true,
                            },
                        },
                        {
                            loader: require.resolve('file-loader'),
                            options: {
                            name: 'static/media/[name].[hash].[ext]',
                            },
                        },
                    ],
                    // 在这些条件中生效
                    issuer: {
                    and: [/\.(ts|tsx|js|jsx|md|mdx)$/],
                    },
                },
                {
                    test: /\.(js|mjs|jsx|ts|tsx)$/,
                    include: paths.appSrc,
                    loader: require.resolve('babel-loader'),
                    options: {
                        // babel-preset-react-app是cra自定义的preset,包括了 JSX, Flow, TypeScript, and some ESnext features
                        customize: require.resolve(
                            'babel-preset-react-app/webpack-overrides'
                        ),
                        presets: [
                            [
                            require.resolve('babel-preset-react-app'),
                            {
                                runtime: hasJsxRuntime ? 'automatic' : 'classic',
                            },
                            ],
                        ],
                        // 一下两个eject后会移除
                        babelrc: false,
                        configFile: false,
                        // 确保 cache identifier的唯一性，eject后会移除
                        cacheIdentifier: getCacheIdentifier(
                            isEnvProduction
                            ? 'production'
                            : isEnvDevelopment && 'development',
                            [
                            'babel-plugin-named-asset-import',
                            'babel-preset-react-app',
                            'react-dev-utils',
                            'react-scripts',
                            ]
                        ),
                        plugins: [
                            isEnvDevelopment &&
                            shouldUseReactRefresh &&
                            require.resolve('react-refresh/babel'),
                        ].filter(Boolean),
                        // babel-loader能将缓存保存在./node_modules/.cache/babel-loader/
                        cacheDirectory: true,
                        cacheCompression: false,
                        compact: isEnvProduction,
                    },
                },
                // 处理其他js
                {
                    test: /\.(js|mjs)$/,
                    exclude: /@babel(?:\/|\\{1,2})runtime/,
                    loader: require.resolve('babel-loader'),
                    options: {
                        ... 同上
                    },
                },
                {
                    test: cssRegex,
                    exclude: cssModuleRegex,
                    use: getStyleLoaders({
                    importLoaders: 1,
                    sourceMap: isEnvProduction
                        ? shouldUseSourceMap
                        : isEnvDevelopment,
                    modules: {
                        mode: 'icss',
                    },
                    }),
                    sideEffects: true,
                },
                {
                    test: cssModuleRegex,
                    use: getStyleLoaders({
                    importLoaders: 1,
                    sourceMap: isEnvProduction
                        ? shouldUseSourceMap
                        : isEnvDevelopment,
                    modules: {
                        mode: 'local',
                        getLocalIdent: getCSSModuleLocalIdent,
                    },
                    }),
                },
                {
                    test: sassRegex,
                    exclude: sassModuleRegex,
                    use: getStyleLoaders(
                    {
                        importLoaders: 3,
                        sourceMap: isEnvProduction
                        ? shouldUseSourceMap
                        : isEnvDevelopment,
                        modules: {
                        mode: 'icss',
                        },
                    },
                    'sass-loader'
                    ),
                    sideEffects: true,
                },
                {
                    test: sassModuleRegex,
                    use: getStyleLoaders(
                    {
                        importLoaders: 3,
                        sourceMap: isEnvProduction
                        ? shouldUseSourceMap
                        : isEnvDevelopment,
                        modules: {
                        mode: 'local',
                        getLocalIdent: getCSSModuleLocalIdent,
                        },
                    },
                    'sass-loader'
                    ),
                },
                // 兜底的'file loader'
                {
                    exclude: [/^$/, /\.(js|mjs|jsx|ts|tsx)$/, /\.html$/, /\.json$/],
                    type: 'asset/resource',
                },
            ],
        },
    ].filter(Boolean),
```

### plugins:

plugins:各种插件，作用见注释

```yaml
plugins: [
    new HtmlWebpackPlugin(
    Object.assign(
        {},
        {
        inject: true,
        template: paths.appHtml,
        },
        isEnvProduction
        ? {
            minify: {
                removeComments: true,
                collapseWhitespace: true,
                removeRedundantAttributes: true,
                useShortDoctype: true,
                removeEmptyAttributes: true,
                removeStyleLinkTypeAttributes: true,
                keepClosingSlash: true,
                minifyJS: true,
                minifyCSS: true,
                minifyURLs: true,
            },
            }
        : undefined
    )
    ),
    isEnvProduction &&
    shouldInlineRuntimeChunk &&
    new InlineChunkHtmlPlugin(HtmlWebpackPlugin, [/runtime-.+[.]js/]),
    // 指定index.html中可以使用的变量，如<link rel="icon" href="%PUBLIC_URL%/favicon.ico">
    new InterpolateHtmlPlugin(HtmlWebpackPlugin, env.raw),
    new ModuleNotFoundPlugin(paths.appPath),
    new webpack.DefinePlugin(env.stringified),
    isEnvDevelopment &&
    shouldUseReactRefresh &&
    new ReactRefreshWebpackPlugin({
        overlay: false,
    }),
    // 大小写敏感，这个插件挺有用的
    isEnvDevelopment && new CaseSensitivePathsPlugin(),
    isEnvProduction &&
    new MiniCssExtractPlugin({
        filename: 'static/css/[name].[contenthash:8].css',
        chunkFilename: 'static/css/[name].[contenthash:8].chunk.css',
    }),
 
    new WebpackManifestPlugin({
        fileName: 'asset-manifest.json',
        publicPath: paths.publicUrlOrPath,
        generate: (seed, files, entrypoints) => {
            const manifestFiles = files.reduce((manifest, file) => {
            manifest[file.name] = file.path;
            return manifest;
            }, seed);
            const entrypointFiles = entrypoints.main.filter(
            fileName => !fileName.endsWith('.map')
            );

            return {
            files: manifestFiles,
            entrypoints: entrypointFiles,
            };
        },
    }),
    // 不打包momentjs中的语言包
    new webpack.IgnorePlugin({
        resourceRegExp: /^\.\/locale$/,
        contextRegExp: /moment$/,
    }),
    // service worker
    isEnvProduction &&
    fs.existsSync(swSrc) &&
    new WorkboxWebpackPlugin.InjectManifest({
        swSrc,
        dontCacheBustURLsMatching: /\.[0-9a-f]{8}\./,
        exclude: [/\.map$/, /asset-manifest\.json$/, /LICENSE/],
        maximumFileSizeToCacheInBytes: 5 * 1024 * 1024,
    }),
    // 改动ts文件触发类型检查
    useTypeScript &&
    new ForkTsCheckerWebpackPlugin({
        async: isEnvDevelopment,
        typescript: {
            typescriptPath: resolve.sync('typescript', {
                basedir: paths.appNodeModules,
            }),
            configOverwrite: {
                compilerOptions: {
                sourceMap: isEnvProduction
                    ? shouldUseSourceMap
                    : isEnvDevelopment,
                skipLibCheck: true,
                inlineSourceMap: false,
                declarationMap: false,
                noEmit: true,
                incremental: true,
                tsBuildInfoFile: paths.appTsBuildInfoFile,
                },
            },
            context: paths.appPath,
            diagnosticOptions: {
                syntactic: true,
            },
            mode: 'write-references',
        },
        issue: {
            include: [
                { file: '../**/src/**/*.{ts,tsx}' },
                { file: '**/src/**/*.{ts,tsx}' },
            ],
            exclude: [
                { file: '**/src/**/__tests__/**' },
                { file: '**/src/**/?(*.){spec|test}.*' },
                { file: '**/src/setupProxy.*' },
                { file: '**/src/setupTests.*' },
            ],
        },
        logger: {
            infrastructure: 'silent',
        },
    }),
    !disableESLintPlugin &&
    new ESLintPlugin({
        extensions: ['js', 'mjs', 'jsx', 'ts', 'tsx'],
        formatter: require.resolve('react-dev-utils/eslintFormatter'),
        eslintPath: require.resolve('eslint'),
        failOnError: !(isEnvDevelopment && emitErrorsAsWarnings),
        context: paths.appSrc,
        cache: true,
        cacheLocation: path.resolve(
            paths.appNodeModules,
            '.cache/.eslintcache'
        ),
        // ESLint class options
        cwd: paths.appPath,
        resolvePluginsRelativeTo: __dirname,
        baseConfig: {
        extends: [require.resolve('eslint-config-react-app/base')],
        rules: {
            ...(!hasJsxRuntime && {
            'react/react-in-jsx-scope': 'error',
            }),
        },
        },
    }),
].filter(Boolean),

};
```

### performance

performance:CRA自带了FileSizeReporter

```vbnet
performance: false,
```

eject命令
-------

eject命令并不复杂，主要也就是是将react-scripts中的文件输出出去，对于eject后不需要的内容，文件中都已经做了标记：

```sql
// @remove-on-eject-end
...
// @remove-on-eject-begin
```

eject过程成中会用正则将这些标记的代码给删除，另外再删除掉react-scripts包，并修改package.json中的几个命令，如`"start": "node scripts/start.js",`

结尾
--

Create React App的源码解析就到这里，后续的文章将会介绍如何自己设计和搭建一个CLI,并不断将其改造完善成一个企业级的脚手架。