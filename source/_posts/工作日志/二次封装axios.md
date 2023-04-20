
---
title: 二次封装axios
tags: []
categories: 
    - - 工作日志
date: 2023-04-13 10:38：00
---
### 封装axios请求
> axios是一个基于promise的http请求库，可以用于浏览器和node环境。
#### 为什仫要对axios进行二次封装
1.封装通用功能：二次封装可以封装通用的功能，比如请求头添加token、统一处理错误信息等，这样可以减少重复的代码，提高开发效率。
2.方便维护：通过二次封装可以将请求和处理逻辑分离，使得代码结构更加清晰，方便维护。
3.接口升级：在接口升级的情况下，如果没有二次封装，那么所有调用该接口的地方都需要进行修改，如果使用了二次封装，那么只需要修改封装的代码即可。
4.更好的可扩展性：通过二次封装，我们可以根据业务需求进行定制化的处理，比如统一添加loading效果、处理特殊的请求参数等。
5.更好的适应性：二次封装可以根据不同的业务场景进行定制，比如不同的请求方式、不同的请求地址等，使得代码更加灵活、适应性更强。
#### 如何实现axios二次封装
我们可以创建一个`request.ts`文件，在该文件中实现axios的二次封装。
- 1.封装 Axios 的基本配置：这包括设置请求的默认 URL、HTTP 方法、请求头、响应数据格式等等。
```javascript
import axios from 'axios';

const instance = axios.create({
  baseURL: 'http://your-api-base-url.com',
  timeout: 1000,
  headers: {
    'Content-Type': 'application/json'
  }
});

export default instance; // 导出axios实例
```

- 2.处理请求参数和响应数据：对于一些通用的数据处理操作，比如请求参数的转换或响应数据的处理，避免在每个请求中都写相同的代码。
  ```javascript
    // 我们可以使用 instance.interceptors.request.use() 方法来添加请求拦截器
    instance.interceptors.request.use(
    config => {
        // 处理请求参数
        config.params = {
        ...config.params,
        api_key: 'xxxxxxxxxxxxxx',
        };
        return config;
    },
    error => {
        // 处理请求错误
        throw error;
    });
    // 类似地，我们也可以使用 instance.interceptors.response.use() 方法来添加响应拦截器
    instance.interceptors.response.use(
    response => {
        // 处理响应数据
        return response.data;
    },
    error => {
        // 处理响应错误
        throw error;
    }
    );
  ```
- 3.处理请求错误：在请求过程中可能会发生错误，比如请求超时或网络错误。针对这些错误，需要进行适当的处理，比如进行重试或提示用户错误信息,可通过拦截器第二个参数配置，可参照第二个例子。
- 4.统一处理请求返回数据格式：如果项目要求返回数据格式一致，可以在封装中统一处理，而不是在每个请求中处理。
    ```javascript
    // 假设与后端约定返回的字段为
    // {
    //   "code": 200,
    //   "message": "",
    //   "data": {}
    // }
    // 我们可以在响应拦截中判断响应数据的 code 字段是否为 200，如果是，就返回 data 字段，否则就抛出一个异常
        instance.interceptors.response.use(
        response => {
            const { data } = response;
            if (data.code === 200) {
            return data.data;
            } else {
            // 处理异常情况
            throw new Error(data.message || '未知错误');
            }
        },
        error => {
            // 处理请求错误
            throw error;
        }
        );
```

- 5.处理接口鉴权：如果项目需要接口鉴权，可以在二次封装中进行处理，统一添加鉴权信息。
  ```javascript
  instance.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token'); // 获取存储在本地的 token
    if (token) {
      config.headers.Authorization = `Bearer ${token}`; // 在请求头中添加鉴权信息
    }
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);
  ```

- 6.处理请求取消：在一些场景下，用户可能需要取消正在进行的请求，比如在搜索框中输入关键字时，用户很可能会多次发送请求，此时需要取消前一次的请求，只保留最后一次请求的结果。

```javascript

    import axios from 'axios';

    const { CancelToken } = axios;
    const source = CancelToken.source();

    const request = axios.create({
    baseURL: process.env.VUE_APP_BASE_API,
    timeout: 5000,
    cancelToken: source.token //将source.token属性作为cancelToken参数传递给axios
    });

    source.cancel('请求已被取消！'); // 取消请求
```
- 7.处理并发请求：在一些场景下，需要同时发送多个请求并等待所有请求都完成后再进行下一步操作，可以使用 Axios 提供的并发请求功能来处理，以便更好地控制网络流量。

    ```javascript
    const instance = axios.create({
    baseURL: 'https://api.example.com',
    timeout: 5000,
    maxContention: 3, // 限制并发请求数量
    });
    ```
    

### 封装api方法
- 创建一个`api.js`文件，并在其中封装api方法
```javascript
import request from './request.js';

export const getUserList = (params) => {
  return request.get('/api/users', {
    params,
  });
};

export const getUserInfo = (id) => {
  return request.get(`/api/users/${id}`);
};

export const addUser = (data) => {
  return request.post('/api/users', data);
};

export const updateUser = (id, data) => {
  return request.put(`/api/users/${id}`, data);
};

export const deleteUser = (id) => {
  return request.delete(`/api/users/${id}`);
};
```
- 使用api方法
  ```javascript
  import { getUserList, addUser, updateUser, deleteUser } from './api';
  getUserList({
    page: 1,
    size: 10,
  })
    .then(data => {
      console.log(data);
    })
    .catch(error => {
      console.error(error);
    });

    addUser({
      name: 'Tom',
      age: 18,
    })
      .then(data => {
        console.log(data);
      })
      .catch(error => {
        console.error(error);
      });

    updateUser(1, {
      name: 'Jerry',
      age: 20,
    })
      .then(data => {
        console.log(data);
      })
      .catch(error => {
        console.error(error);
      });

    deleteUser(1)
      .then(data => {
        console.log(data);
      })
      .catch(error => {
        console.error(error);
      });
  ```