#### entry-server.js
```javascript
import createApp from "./app.js";

export default (context) => {
    return new Promise((reslove,reject) => {
        let {url} = context;
        let {app,router} = createApp(context);
        router.push(url);
        router.onReady(() => {
            let matchedComponents = router.getMatchedComponents();
            if(!matchedComponents.length){
                return reject();
            }
            reslove(app);
        },reject)
    })
}
```

#### webpack.server.conf.js
```javascript
const webpack = require("webpack");
const merge = require("webpack-merge");
const base = require("./webpack.base.conf");
const webapckNodeExternals = require("webpack-node-externals");


const vueSSRServerPlugin = require("vue-server-renderer/server-plugin");

module.exports = merge(base,{
    //  告知webpack，需要在node端运行
    target:"node",
    entry:"./src/entry-server.js",
    devtool:"source-map",
    output:{
        filename:'server-buldle.js',
        libraryTarget: "commonjs2"
    },
    externals:[
        webapckNodeExternals()
    ],
    plugins:[
        new webpack.DefinePlugin({
            'process.env.NODE_ENV':'"development"',
            'process.ent.VUE_ENV': '"server"'
        }),
        new vueSSRServerPlugin()
    ]
});
```

#### build/dev-server.js
```javascript
const serverConf = require("./webpack.server.conf");
const webpack = require("webpack");
const fs = require("fs");
const path = require("path");
//  读取内存中的.json文件
//  这个模块需要手动安装
const Mfs = require("memory-fs");
const axios = require("axios");

module.exports = (cb) => {
    const webpackComplier = webpack(serverConf);
    var mfs = new Mfs();
    
    webpackComplier.outputFileSystem = mfs;
    
    webpackComplier.watch({},async (error,stats) => {
        if(error) return console.log(error);
        stats = stats.toJson();
        stats.errors.forEach(error => console.log(error));
        stats.warnings.forEach(warning => console.log(warning));
        //  获取server bundle的json文件
        let serverBundlePath = path.join(serverConf.output.path,'vue-ssr-server-bundle.json');
        let serverBundle = JSON.parse(mfs.readFileSync(serverBundlePath,"utf-8"));
        //  获取client bundle的json文件
        let clientBundle = await axios.get("http://localhost:8082/vue-ssr-client-manifest.json");
        //  获取模板
        let template = fs.readFileSync(path.join(__dirname,"..","index.html"),"utf-8");
        cb && cb(serverBundle,clientBundle,template);
    })
};
```

#### server.js
```javascript
const devServer = require("./build/dev-server.js");
const express = require("express");
const app = express();
const vueRender = require("vue-server-renderer");

app.get('*',(request,respones) => {
    respones.status(200);
    respones.setHeader("Content-Type","text/html;charset-utf-8;");
    devServer((serverBundle,clientBundle,template) => {
        let render = vueRender.createBundleRenderer(serverBundle,{
            template,
            clientManifest:clientBundle.data,
            //  每次创建一个独立的上下文
            renInNewContext:false
        }); 
        render.renderToString({
            url:request.url
        }).then((html) => {
            respones.end(html);
        }).catch(error => {
          respones.end(JSON.stringify(error));
        });
    });
})

app.listen(5001,() => {
    console.log("服务已开启")
});
```
