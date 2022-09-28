# Vue商城后台管理系统

## 项目介绍
- 该项目是一个商城后台管理系统的前端项目，基于Vue+Element-UI实现。主要包括用户登录、用户管理、权限管理、商品管理、订单管理、数据管理六大模块。

## 项目演示
- 在线演示地址：https://monster-crj.github.io/management-system/

## 技术选型
| 技术              | 说明                  | 官网                                                         |
| ----------------- | --------------------- | ------------------------------------------------------------ |
| Vue               | 前端框架              | [https://vuejs.org/](https://vuejs.org/)                     |
| VueRouter         | 路由框架              | [https://router.vuejs.org/](https://router.vuejs.org/)       |
| Element-UI        | 前端UI框架            | [https://element.eleme.io/](https://element.eleme.io/)       |
| Axios             | 前端HTTP框架          | [https://github.com/axios/axios](https://github.com/axios/axios) |
| Echarts           | 基于JavaScript的开源可视化图表库 | [https://echarts.apache.org/](https://echarts.apache.org/) |

## 项目起步
### 一、创建项目

### 二、项目结构
```
└── /src/            # 源码目录
  ├── /assets/       # 组件静态资源
  ├── /components/   # 公共组件
  ├── /plugins/      # 插件
  ├── /router/       # 路由配置
  ├── /views/        # 路由组件
  ├── /utils/        # 工具
  ├── App.vue        # 组件入口
  └── main.js        # 程序入口
```
### 三、路由设计

#### 1.路由结构
| 路径          | 组件  | 嵌套级别 |
| ------------- | ------ | ------ |
| /login        | 登录    | 1级 |
| /home         | 系统主页 | 1级 |
| /home/welcome | 欢迎页  | 2级 |
| /home/users   | 用户列表 | 2级 |
| /home/rights  | 权限列表 | 2级 |
| /home/roles   | 角色列表 | 2级 |
| //home/goods  | 商品列表 | 2级 |
| /home/params  | 商品参数 | 2级 |
| /home/orders  | 订单列表 | 2级 |
| /home/reports | 数据报表 | 2级 |

#### 2.设置全局路由守卫
- 用户如果没有登录，通过url访问特定页面，强制跳转到登录页面
```js
router.beforeEach((to, form, next) => {
  if (to.path === '/login') {
    next()
  }
  else {
    const token = window.sessionStorage.getItem('token')
    if (!token) {
      next('/login')
    }
    else {
      next()
    }
  }
})
```

### 四、请求工具
- 对axios进行二次封装，设置请求拦截器和响应拦截器，并将axios挂载到vue原型上
- 请求拦截器中，将本地存储的token携带在请求头中，请求时开启loading动画
- 响应拦截器中，响应时关闭loading动画
```js
let loadingInstance
axios.interceptors.request.use((config) => {
  config.headers.Authorization = window.sessionStorage.getItem('token')
  loadingInstance = Vue.prototype.$loading({
    background: 'transparent',
  })
  return config
})
axios.interceptors.response.use((response) => {
  loadingInstance.close()
  return response
})
```

### 五、样式处理
- 定义全局样式
```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body,
#app {
  height: 100%;
}

.el-breadcrumb {
  margin-bottom: 20px;
  font-size: 15px;
}
```


## 功能模块

### 一、登录模块

#### 1.用户登录
- 用户登录前先进行表单校验，再发送登录请求，根据后端返回的状态码调用操作成功或者失败的消息提示框；请求成功后将token存储到本地，跳转系统主页面
##### 1.1 组件
```html
      <el-form :model="form" :rules="rule" ref="form">
        <el-form-item prop="username">
          <el-input
            prefix-icon="el-icon-user-solid"
            v-model="form.username"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            type="password"
            prefix-icon="el-icon-lock"
            v-model="form.password"
          ></el-input>
        </el-form-item>
        <el-form-item class="button">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
```
##### 1.2 校验规则
```js
      rule: {
        username: [
          {
            required: true,
            pattern: /^[a-zA-Z][a-zA-Z0-9_]{4,15}$/,
            message:
              "请输入正确用户名，要求为字母开头，长度为5-16位，字母数字下划线组合",
            trigger: "blur",
          },
        ],
        password: [
          {
            required: true,
            min: 6,
            max: 18,
            message: "请输入正确密码，要求长度为6-18位",
            trigger: "blur",
          },
        ],
      }
```
##### 1.3 实现代码
```js
    login() {
      this.$refs.form.validate(async (valid) => {
        if (!valid) {
          return;
        } else {
          const { data: res } = await this.$http.post("login", this.form);
          // console.log(res);
          if (res.meta.status !== 200) {
            this.$message.error("登录失败");
          } else {
            this.$message.success("登录成功");
            window.sessionStorage.setItem("token", res.data.token);
            this.$router.push("/home");
          }
        }
      });
    }
```

#### 2.用户退出
- 用户退出时，必须先销毁本地存储的token，再跳转到登录页面
```js
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    }
```

### 二、用户管理模块

#### 1.用户列表
- 在生命周期created中执行
```js
  created() {
    this.getUserList();
  }
```
##### 1.1 组件
```html
      <el-table :data="userList">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="用户名"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="手机号"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="openEditDialog(scope.row.id)"
              >修改</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              @click="openSetDialog(scope.row.id)"
              size="mini"
              >设置</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              @click="openDeleteDialog(scope.row.id)"
              size="mini"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
```
##### 1.2 实现代码
```js
    async getUserList() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo,
      });
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("获取失败");
      } else {
        this.userList = res.data.users;
        this.total = res.data.total;
      }
    }
```

#### 2.用户分页显示

##### 2.1 组件
```html
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="queryInfo.pagenum"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="10"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
```
##### 2.2 实现代码
```js
    handleCurrentChange() {
      this.getUserList();
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val;
      this.queryInfo.pagenum = 1;
      this.getUserList();
    }
```

#### 3.查找用户

##### 3.1 组件
```html
          <el-input
            placeholder="查找用户"
            v-model="queryInfo.query"
            clearable
            @clear="search"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="search"
            ></el-button>
          </el-input>
```
##### 3.2 实现代码
```js
search() {
      this.queryInfo.pagenum = 1;
      this.getUserList();
    }
```

#### 4.添加用户
- 调用dialog组件显示添加用户对话框，注意在对话框关闭后重置表单数据；添加用户时，先进行表单校验，再发送请求，添加成功后重新获取用户列表
##### 4.1 组件
```html
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        @close="resetAddForm"
      >
        <el-form
          :model="addUserInfo"
          :rules="addRule"
          ref="addForm"
          label-width="80px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addUserInfo.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input type="password" v-model="addUserInfo.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addUserInfo.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="addUserInfo.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>
```
##### 4.2 校验规则
```js
      addRule: {
        username: [
          {
            required: true,
            pattern: /^[a-zA-Z][a-zA-Z0-9_]{4,15}$/,
            message:
              "请输入正确用户名，要求为字母开头，长度为5-16位，字母数字下划线组合",
            trigger: "blur",
          },
        ],
        password: [
          {
            required: true,
            min: 6,
            max: 18,
            message: "请输入正确密码，要求长度为6-18位",
            trigger: "blur",
          },
        ],
        email: [
          {
            required: true,
            pattern: /\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}/,
            message: "请输入正确邮箱",
            trigger: "blur",
          },
        ],
        mobile: [
          {
            required: true,
            pattern: /0?(13|14|15|18|17)[0-9]{9}/,
            message: "请输入正确手机号",
            trigger: "blur",
          },
        ],
      }
```
##### 4.3 实现代码
```js
    resetAddForm() {
      this.$refs.addForm.resetFields();
    },
    addUser() {
      this.$refs.addForm.validate(async (valid) => {
        if (!valid) {
          return;
        } else {
          const { data: res } = await this.$http.post(
            "users",
            this.addUserInfo
          );
          // console.log(res);
          if (res.meta.status !== 201) {
            this.$message.error("添加失败");
          } else {
            this.$message.success("添加成功");
            this.addDialogVisible = false;
            this.getUserList();
          }
        }
      });
    }
```

#### 5.修改用户信息
- 根据用户ID获取当前用户的信息，渲染在对话框上，注意在对话框关闭后重置表单数据；修改用户信息时，先进行表单校验，再发送请求，修改成功后重新获取用户列表
##### 5.1 组件
```html
      <el-dialog
        title="修改用户信息"
        :visible.sync="editDialogVisible"
        @close="resetEditForm"
      >
        <el-form
          :model="userInfo"
          :rules="editRule"
          label-width="80px"
          ref="editForm"
        >
          <el-form-item label="用户名">
            <el-input :value="userInfo.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="userInfo.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="userInfo.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
```
##### 5.2 校验规则
```js
      editRule: {
        email: [
          {
            required: true,
            pattern: /\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}/,
            message: "请输入正确邮箱",
            trigger: "blur",
          },
        ],
        mobile: [
          {
            required: true,
            pattern: /0?(13|14|15|18|17)[0-9]{9}/,
            message: "请输入正确手机号",
            trigger: "blur",
          },
        ],
      }
```
##### 5.3 实现代码
```js
    async queryUser(id) {
      const { data: res } = await this.$http.get(`users/${id}`);
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("查询失败");
      } else {
        this.userInfo = res.data;
      }
    },
   openEditDialog(id) {
      this.queryUser(id);
      this.editDialogVisible = true;
    },
    resetEditForm() {
      this.$refs.editForm.resetFields();
    },
    editUser() {
      this.$refs.editForm.validate(async (valid) => {
        if (!valid) {
          return;
        } else {
          const { data: res } = await this.$http.put(
            `users/${this.userInfo.id}`,
            this.userInfo
          );
          // console.log(res);
          if (res.meta.status !== 200) {
            this.$message.error("修改失败");
          } else {
            this.$message.success("修改成功");
            this.editDialogVisible = false;
            this.getUserList();
          }
        }
      });
    }
```

#### 6.设置用户角色
- 根据用户ID获取当前用户信息和角色列表，将用户信息和角色列表渲染在对话框上；设置成功后重新获取用户列表
##### 6.1 组件
```html
      <el-dialog title="设置用户角色" :visible.sync="setDialogVisible">
        <el-form label-width="80px">
          <el-form-item label="用户名">
            <el-input :value="userInfo.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="选择角色">
            <el-select v-model="roleId" placeholder="请选择">
              <el-option
                v-for="item in roleList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="setRole">确 定</el-button>
        </span>
      </el-dialog>
```
##### 6.2 实现代码
```js
    async getRoleList() {
      const { data: res } = await this.$http.get("roles");
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("获取失败");
      } else {
        this.roleList = res.data;
      }
    },
    openSetDialog(id) {
      this.queryUser(id);
      this.getRoleList();
      this.setDialogVisible = true;
    },
    async setRole() {
      if (!this.roleId) {
        this.$message.error("请选择设置角色");
      } else {
        const { data: res } = await this.$http.put(
          `users/${this.userInfo.id}/role`,
          {
            rid: this.roleId,
          }
        );
        // console.log(res);
        if (res.meta.status !== 200) {
          this.$message.error("设置失败");
        } else {
          this.$message.success("设置成功");
          this.setDialogVisible = false;
          this.getUserList();
        }
      }
    }
```

#### 7.删除用户
- 调用全局方法$confirm显示确认消息框，先进行确认操作后，再发送删除请求，删除成功后重新获取用户列表
```js
    openDeleteDialog(id) {
      // console.log(id);
      this.$confirm("此操作将永久删除该用户, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.removeUser(id);
        })
        .catch(() => {
          this.$message.info("已取消删除");
        });
    },
    async removeUser(id) {
      const { data: res } = await this.$http.delete(`users/${id}`);
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("删除失败");
      } else {
        this.$message.success("删除成功");
        this.getUserList();
      }
    }
```

### 三、权限管理模块

#### 1.权限列表
- 同用户列表，注意请求tree型数据，通过table组件中的tree-props属性渲染嵌套数据
```html
      <el-table
        :data="rightList"
        row-key="id"
        :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
        default-expand-all
      >
        <el-table-column prop="authName" label="全部权限"></el-table-column>
        <el-table-column prop="path" label="权限字典"></el-table-column>
      </el-table>
```

#### 2.角色列表
- 同用户列表

#### 3.添加角色
- 同添加用户

#### 4.修改角色信息
- 同修改用户信息

#### 5.设置角色权限
```html
      <el-dialog
        title="设置角色权限"
        :visible.sync="setDialogVisible"
        @close="nodeKey = []"
      >
        <el-tree
          :data="rightList"
          :props="defaultProps"
          show-checkbox
          default-expand-all
          node-key="id"
          :default-checked-keys="nodeKey"
          ref="tree"
        ></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="setRights">确 定</el-button>
        </span>
      </el-dialog>
```
```js
    openSetDialog(role) {
      this.queryRole(role.id);
      this.getRightList();
      this.getNodeKey(role, this.nodeKey);
      this.setDialogVisible = true;
    },
    getNodeKey(node, arr) {
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach((item) => {
        this.getNodeKey(item, arr);
      });
    },
    async setRights() {
      const keys = [
        ...this.$refs.tree.getCheckedKeys(),
        ...this.$refs.tree.getHalfCheckedKeys(),
      ];
      const rids = keys.join();
      const { data: res } = await this.$http.post(
        `roles/${this.roleInfo.roleId}/rights`,
        {
          rids,
        }
      );
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("设置失败");
      } else {
        this.$message.success("设置成功");
        this.setDialogVisible = false;
        this.getRoleList();
      }
    }
```
#### 6.删除角色
- 同删除用户

### 四、商品管理模块

#### 1.商品列表
- 同用户列表

#### 2.商品分页显示
- 同用户分页显示

#### 3.查找商品
- 同查找用户

#### 4.添加商品

##### 4.1 组件
```html
      <el-form
        :model="addGoodsInfo"
        :rules="addRule"
        ref="addForm"
        label-width="80px"
      >
        <el-tabs tab-position="left" value="info">
          <el-tab-pane label="基本信息" name="info">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodsInfo.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model.number="addGoodsInfo.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model.number="addGoodsInfo.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model.number="addGoodsInfo.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addGoodsInfo.goods_cat"
                :options="cateList"
                :props="props"
              >
              </el-cascader>
            </el-form-item>
            <el-form-item label="商品介绍" prop="goods_introduce">
              <el-input v-model="addGoodsInfo.goods_introduce"> </el-input>
            </el-form-item>
            <el-button type="primary" @click="addGoods">添加商品</el-button>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="img"
            ><el-upload
              :action="uploadUrl"
              :headers="headers"
              :on-success="handleSuccess"
              :on-remove="handleRemove"
              list-type="picture"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload></el-tab-pane
          >
        </el-tabs>
      </el-form>
```
##### 4.2 实现代码
```js
    // 获取分类列表
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("获取失败");
      } else {
        this.cateList = res.data;
      }
    },

    // 添加商品
    addGoods() {
      this.$refs.addForm.validate(async (valid) => {
        if (!valid) {
          return;
        } else {
          if (Array.isArray(this.addGoodsInfo.goods_cat)) {
            this.addGoodsInfo.goods_cat = this.addGoodsInfo.goods_cat.join(",");
          }
          const { data: res } = await this.$http.post(
            "goods",
            this.addGoodsInfo
          );
          // console.log(res);
          if (res.meta.status !== 201) {
            this.$message.error("添加失败");
          } else {
            this.$message.success("添加成功");
            this.$router.push("/home/goods");
          }
        }
      });
    },

    // 上传图片
    handleSuccess(res) {
      const info = { pic: res.data.tmp_path };
      this.addGoodsInfo.pics.push(info);
    },
    handleRemove(file) {
      const path = file.response.data.tmp_path;
      const index = this.addGoodsInfo.pics.findIndex((item) => {
        item.pcis === path;
      });
      this.addGoodsInfo.pics.splice(index);
    }
```

#### 5.查看商品图片
- 根据商品ID获取当前商品的信息，将图片数据渲染到对话框上

#### 6.修改商品信息
- 同修改用户信息

#### 7.删除商品
- 同删除用户

#### 8.商品参数
- 获取商品分类列表，渲染到cascader组件上；通过cascader组件选择商品分类，根据所选的商品分类ID获取商品参数渲染到tabs组件的表格上，当切换参数类型时，重新获取商品参数
##### 8.1 组件
```html
      <el-cascader
        v-model="value"
        :options="cateList"
        :props="props"
        @change="getParams"
        placeholder="请选择商品分类"
      >
      </el-cascader>
```
``` html
      <el-tabs v-model="activeName" @tab-click="getParams" stretch>
        <el-tab-pane label="固定参数" name="only">
          <el-table :data="params">
            <el-table-column
              prop="attr_name"
              label="参数名称"
              width="300px"
            ></el-table-column>
            <el-table-column prop="attr_vals" label="参数值"></el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="变化参数" name="many">
          <el-table :data="params">
            <el-table-column
              prop="attr_name"
              label="参数名称"
              width="300px"
            ></el-table-column>
            <el-table-column prop="attr_vals" label="参数值"></el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
```
##### 8.2 实现代码
```js
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("获取失败");
      } else {
        this.cateList = res.data;
      }
    }
```
```js
    async getParams() {
      const { data: res } = await this.$http.get(
        `categories/${this.value[2]}/attributes`,
        {
          params: { sel: this.activeName },
        }
      );
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("获取失败");
      } else {
        this.params = res.data;
        // console.log(this.params);
      }
    }
``` 

### 五、订单管理模块

#### 1.订单列表
- 同用户列表

#### 2.订单分页显示
- 同用户分页显示

#### 3.查找订单
- 同查找用户

#### 4.查看订单物流信息
- 根据订单ID获取当前订单物流信息，渲染到对话框中的timeline组件上
##### 4.1 组件
```html
    <el-dialog title="查看物流信息" :visible.sync="dialogVisible">
      <el-timeline>
        <el-timeline-item
          v-for="(item, index) in transInfo"
          :key="index"
          :timestamp="item.time"
        >
          {{ item.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
```
##### 4.2 实现代码
```js
    openDiglog(id) {
      this.querytransportation(id);
      this.dialogVisible = true;
    },
    async querytransportation(id) {
      const { data: res } = await this.$http.get(`kuaidi/${id}`);
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("查询失败");
      } else {
        this.transInfo = res.data;
      }
    }
```

### 六、数据管理模块
- 在生命周期mounted中执行
- 获取报表数据，渲染在echars实例上
```js
  mounted() {
    this.chart();
  }
```
```js
    async chart() {
      // 基于准备好的dom，初始化echarts实例
      let myChart = echarts.init(document.getElementById("chart"));
      // 指定图表的配置项和数据
      let option;
      const { data: res } = await this.$http.get("reports/type/1");
      // console.log(res);
      if (res.meta.status !== 200) {
        this.$message.error("获取失败");
      } else {
        option = res.data;
      }
      // 使用刚指定的配置项和数据显示图表。
      myChart.setOption(option);
    }
```

