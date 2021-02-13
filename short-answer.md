### 一. 简答题

#### 1.说说 application/json 和 application/x-www-form-urlencoded 二者之间的区别？

答：

- application/json
  - 数据传输模式是以 json 格式字符串进行传输
  - 前端人员不用担心传参复杂度，只要符合 json 格式就行
- application/x-www-form-urlencoded
  - 数据传输会被编码为键值对
  - 浏览器都支持

#### 2. 说一说在前端这块，角色管理你是如何设计的？

答：

- 角色管理

  | 功能     | 字段                     | 描述                                     |
  | -------- | ------------------------ | ---------------------------------------- |
  | 查询     | 角色名称                 | 通过角色名称查询角色列表                 |
  | 新建     | 角色名称，角色编码，描述 | 通过填写角色名称，角色编码，描述新建角色 |
  | 编辑     | 角色名称，角色编码，描述 | 通过编辑角色名称，角色编码，描述编辑角色 |
  | 删除     | 角色 id                  | 通过角色 id 来删除对应角色               |
  | 分配菜单 | 菜单 id 数组             | 通过绑定菜单 id，来实现角色和菜单的绑定  |

- 菜单管理

  | 功能     | 字段                                                         | 描述                                                         |
  | -------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
  | 查询     | 菜单名称                                                     | 通过菜单名称查询菜单列表                                     |
  | 新建     | 菜单名称，菜单路径，上级菜单，描述，前端图标，是否显示，排序 | 通过填写菜单名称，菜单路径，上级菜单，描述，前端图标，是否显示，排序新建菜单 |
  | 编辑     | 菜单名称，菜单路径，上级菜单，描述，前端图标，是否显示，排序 | 通过编辑菜单名称，菜单路径，上级菜单，描述，前端图标，是否显示，排序编辑菜单 |
  | 删除     | 菜单 id                                                      | 通过菜单 id 来删除对应菜单                                   |
  | 分配资源 | 资源 id 数组                                                 | 通过绑定资源 id，来实现菜单和资源的绑定                      |

- 资源管理

  | 功能 | 字段                               | 描述                                               |
  | ---- | ---------------------------------- | -------------------------------------------------- |
  | 查询 | 资源名称，资源路径，资源分类       | 通过资源名称，资源路径，资源分类查询资源列表       |
  | 新建 | 资源名称，资源路径，资源分类，描述 | 通过填写资源名称，资源路径，资源分类，描述新建资源 |
  | 编辑 | 资源名称，资源路径，资源分类，描述 | 通过编辑资源名称，资源路径，资源分类，描述编辑资源 |
  | 删除 | 资源 id                            | 通过资源 id 来删除对应资源                         |

- 资源分类管理

  | 功能 | 字段           | 描述                               |
  | ---- | -------------- | ---------------------------------- |
  | 查询 | 分类名称       | 通过分类名称查询资源分类列表       |
  | 新建 | 分类名称，排序 | 通过填写分类名称，排序新建资源分类 |
  | 编辑 | 分类名称，排序 | 通过编辑分类名称，排序编辑资源分类 |
  | 删除 | 分类 id        | 通过分类 id 来删除对应资源分类     |

#### 3. @vue/cli 跟 vue-cli 相比，@vue/cli 的优势在哪？

答：

- @vue/cli 将大量的配置文件工作在内部处理，让开发者配置少量的配置文件，更加简单易用
- @vue/cli 目录结构调整，更加清晰明了
- @vue/cli 拥有 @vue/cli GUI，可以通过可视化界面的方式来配置项目

#### 4. 详细讲一讲生产环境下前端项目的自动化部署的流程？

答：

- 环境准备

  - Linux 服务器
  - 把代码提交到 Github 远程仓库

- 配置 Github Access Token

  - 生成：点击用户选择 settings => Developer settings => Personal access tokens
  - 配置到项目的 Secrets 中：github项目仓库中 settings => secrets

- 配置 Github Actions 执行脚本

  - 在项目根目录创建 .github/workflows 目录

  - 新建 main.yml 文件

    ```
    name: Publish And Deploy
    on:
      push:
        tags:
          - 'v*'
    
    jobs:
      build-and-deploy:
        runs-on: ubuntu-latest
        steps:
    
        # 下载源码
        - name: Checkout
          uses: actions/checkout@master
    
        # 打包构建
        - name: Build
          uses: actions/setup-node@master
        - run: npm install
        - run: npm run build
        - run: tar -zcvf release.tgz .nuxt static nuxt.config.js package.json package-lock.json pm2.config.json
    
        # 发布 Release
        - name: Create Release
          id: create_release
          uses: actions/create-release@master
          env:
            GITHUB_TOKEN: ${{ secrets.TOKEN }}
          with:
            tag_name: ${{ github.ref }}
            release_name: Release ${{ github.ref }}
            draft: false
            prerelease: false
    
        # 上传构建结果到 Release
        - name: Upload Release Asset
          id: upload-release-asset
          uses: actions/upload-release-asset@master
          env:
            GITHUB_TOKEN: ${{ secrets.TOKEN }}
          with:
            upload_url: ${{ steps.create_release.outputs.upload_url }}
            asset_path: ./release.tgz
            asset_name: release.tgz
            asset_content_type: application/x-tgz
    
        # 部署到服务器
        - name: Deploy
          uses: appleboy/ssh-action@master
          with:
            host: ${{ secrets.HOST }}
            username: ${{ secrets.USERNAME }}
            password: ${{ secrets.PASSWORD }}
            port: ${{ secrets.PORT }}
            script: |
              cd /home/real-world-nuxtjs
              wget https://github.com/kangjinyuan/real-world-nuxtjs/releases/latest/download/release.tgz -O release.tgz
              tar zxvf release.tgz
              npm install --production
              pm2 reload pm2.config.json
    
    ```

    

- 配置 PM2 配置文件

- 查看自动部署状态

#### 5. 你在开发过程中，遇到过哪些问题，又是怎样解决的？请讲出两点。

答：

| 问题                   | 解决办法                                           |
| ---------------------- | -------------------------------------------------- |
| 请求接口跨域           | 项目根目录新建 vue.config.js，在此文件中配置 proxy |
| 路由页面无登录禁止访问 | 在 router/index.ts 中 设置路由守卫                 |

#### 6. 针对新技术，你是如何过渡到项目中？

答：针对新技术，对于项目新需求，在确保旧的功能不受影响的前提下直接引入，以旧功能代码为模板，完成新技术的迁移之后，稳定之后，慢慢移除旧功能