# 技术选型

## 前端

vue
UI框架：elementplus
语言：js

## 后端

语言：java
框架：springboot
orm：mybatis-flex
权限框架：sa-token
OSS对象存储：七牛云

# 环境要求
## 操作系统

支持多端，跨平台，例如windows，linux

## 游览器

chrome内核游览器

## 软件环境

### 前端

nodejs版本：20.13.1以上
vue版本：vue3

### 后端

JDK：该项目使用的是JDK21


# 项目运行
## 开发环境运行(本地)

### 前端

在项目根目录下，在命令行下运行

```shell
npm run dev
```

或者

```shell
npm run serve
```

在集成开发环境中打开即可（例如vscdoe，webstorm）
### 后端

在集成开发环境中运行即可（例如idea，eclipse）

## 线上环境配置(以nginx为例)

前端项目执行build命令，将dist目录放在nginx的html目录下。后端打成jar包即可。

# 接口文档

以内嵌到项目中

![[屏幕截图 2024-10-02 213739.jpg]]


# 项目部分功能展示

## 登录
![[屏幕截图 2024-10-02 214015.jpg]]

### 登录说明

登录采用双token。前端传入用户名密码后，后端通过数据库后，若用户名密码正确，返回给前端accessToken，refreshToken，accessToken过期时间，每次请求都会携带accessToken，后端校验token，检验通过才能访问接口。若accessToken过期，refreshToken没过期，后端将使用refreshToken来生成一个新的accessToken给前端，以此达到无感刷新。若accessToken和refreshToken均过期，则登录过期，跳转至登录页面

### 核心代码

```java
public LoginUser login(LoginForm loginForm) {  
    SystemUserDO systemUserDO = systemUserMapper.selectOneByUsernamePassword(loginForm.getUsername(), loginForm.getPassword());  
    if (null == systemUserDO){  
        throw  new BusinessException(USER_USERNAME_OR_PASSWORD_ERROR);  
    }    if (CommonType.DISABLE.getCode().equals(systemUserDO.getStatus())){  
        throw  new BusinessException(USER_ACCOUNT_DISABLE);  
    }    return buildLoginUser(systemUserDO);  
}  
  
public LoginUser buildLoginUser(SystemUserDO systemUserDO){  
    Integer id = systemUserDO.getId();  
    String token = SaTempUtil.createToken(systemUserDO , 60 * 60 * 24);  
    LoginUser loginUser = new LoginUser();  
    loginUser.setUserId(id);  
    loginUser.setRoleList(systemRoleMapper.getRoleCodeListByUserId(id));  
    loginUser.setRefreshToken(token);  
    loginUser.setUsername(systemUserDO.getUsername());  
    loginUser.setNickName(systemUserDO.getNickName());  
    loginUser.setAvatar(systemUserDO.getAvatar());  
  
    LoginUserUtils.loginByDevice(loginUser, CommonType.LoginType.PC);  
    loginUser.setAccessToken(StpUtil.getTokenValue());  
    loginUser.setExpires(Instant.now().plusSeconds(StpUtil.getTokenTimeout()).toEpochMilli());  
    return loginUser;  
}
```

```java
public class LoginUserUtils {  
  
  
    public static final String LOGIN_USER_KEY = "loginUser";  
    public static final String USER_KEY = "userId";  
  
    public static void loginByDevice(LoginUser loginUser, CommonType.LoginType loginType) {  
        SaStorage storage = SaHolder.getStorage();  
        storage.set(LOGIN_USER_KEY, loginUser);  
        storage.set(USER_KEY, loginUser.getUserId());  
        SaLoginModel model = new SaLoginModel();  
        model.setDevice(loginType.getLoginType());  
        //accessToken有效期1个小时  
        model.setTimeout(60 * 60);  
        StpUtil.login(loginUser.getUserId(), model.setExtra(USER_KEY, loginUser.getUserId()));  
        StpUtil.getTokenSession().set(LOGIN_USER_KEY, loginUser);  
    }  
  
    public static LoginUser getLoginUser() {  
        LoginUser loginUser = (LoginUser) SaHolder.getStorage().get(LOGIN_USER_KEY);  
        if (loginUser != null) {  
            return loginUser;  
        }        SaSession session = StpUtil.getTokenSession();  
        if (null==session) {  
            return null;  
        }        loginUser = (LoginUser) session.get(LOGIN_USER_KEY);  
        if (loginUser == null){  
            throw new BusinessException(UNAUTHORIZED);  
        }        SaHolder.getStorage().set(LOGIN_USER_KEY, loginUser);  
        return loginUser;  
    }  
  
}
```

```ts
import Axios, {  
  type AxiosInstance,  
  type AxiosRequestConfig,  
  type CustomParamsSerializer  
} from "axios";  
import type {  
  PureHttpError,  
  RequestMethods,  
  PureHttpResponse,  
  PureHttpRequestConfig  
} from "./types.d";  
import { stringify } from "qs";  
import NProgress from "../progress";  
import { getToken, formatToken } from "@/utils/auth";  
import { useUserStore, useUserStoreHook } from "@/store/modules/user";  
import type { CommonResult } from "@/api/CommonResult";  
import { ElMessage } from "element-plus";  
  
// 相关配置请参考：www.axios-js.com/zh-cn/docs/#axios-request-config-1  
const defaultConfig: AxiosRequestConfig = {  
  baseURL: "/api",  
  // 请求超时时间  
  timeout: 10000,  
  headers: {  
    Accept: "application/json, text/plain, */*",  
    "Content-Type": "application/json",  
    "X-Requested-With": "XMLHttpRequest"  
  },  
  paramsSerializer: {  
    serialize: stringify as unknown as CustomParamsSerializer  
  }  
};  
  
class PureHttp {  
  constructor() {  
    this.httpInterceptorsRequest();  
    this.httpInterceptorsResponse();  
  }  
  /** `token`过期后，暂存待执行的请求 */  
  private static requests = [];  
  
  /** 防止重复刷新`token` */  
  private static isRefreshing = false;  
  
  /** 初始化配置对象 */  
  private static initConfig: PureHttpRequestConfig = {};  
  
  /** 保存当前`Axios`实例对象 */  
  private static axiosInstance: AxiosInstance = Axios.create(defaultConfig);  
  
  /** 重连原始请求 */  
  private static retryOriginalRequest(config: PureHttpRequestConfig) {  
    return new Promise(resolve => {  
      PureHttp.requests.push((token: string) => {  
        config.headers["Authorization"] = formatToken(token);  
        resolve(config);  
      });    });  }  
  /** 请求拦截 */  
  private httpInterceptorsRequest(): void {  
    PureHttp.axiosInstance.interceptors.request.use(  
      async (config: PureHttpRequestConfig): Promise<any> => {  
        // 开启进度条动画  
        NProgress.start();  
        // 优先判断post/get等方法是否传入回调，否则执行初始化设置等回调  
        if (typeof config.beforeRequestCallback === "function") {  
          config.beforeRequestCallback(config);  
          return config;  
        }        if (PureHttp.initConfig.beforeRequestCallback) {  
          PureHttp.initConfig.beforeRequestCallback(config);  
          return config;  
        }        /** 请求白名单，放置一些不需要`token`的接口（通过设置请求白名单，防止`token`过期后再请求造成的死循环问题） */  
        const whiteList = ["/refreshToken", "/login"];  
        return whiteList.some(url => config.url.endsWith(url))  
          ? config  
          : new Promise(resolve => {  
              const data = getToken();  
              if (data) {  
                const now = new Date().getTime();  
                const expired = parseInt(data.expires) - now <= 0;  
                if (expired) {  
                  if (!PureHttp.isRefreshing) {  
                    PureHttp.isRefreshing = true;  
                    // token过期刷新  
                    useUserStoreHook()  
                      .handRefreshToken({ refreshToken: data.refreshToken })  
                      .then(res => {  
                        const token = res.accessToken;  
                        config.headers["Authorization"] = formatToken(token);  
                        PureHttp.requests.forEach(cb => cb(token));  
                        PureHttp.requests = [];  
                      })                      .finally(() => {  
                        PureHttp.isRefreshing = false;  
                      });                  }                  resolve(PureHttp.retryOriginalRequest(config));  
                } else {  
                  config.headers["Authorization"] = formatToken(  
                    data.accessToken  
                  );  
                  resolve(config);  
                }              } else {  
                resolve(config);  
              }            });      },      error => {  
        return Promise.reject(error);  
      }    );  }  
  /** 响应拦截 */  
  private httpInterceptorsResponse(): void {  
    const instance = PureHttp.axiosInstance;  
    instance.interceptors.response.use(  
      (response: PureHttpResponse) => {  
        const $config = response.config;  
        // 关闭进度条动画  
        NProgress.done();  
        // 优先判断post/get等方法是否传入回调，否则执行初始化设置等回调  
        if (typeof $config.beforeResponseCallback === "function") {  
          $config.beforeResponseCallback(response);  
          return response.data;  
        }        if (PureHttp.initConfig.beforeResponseCallback) {  
          PureHttp.initConfig.beforeResponseCallback(response);  
          return response.data;  
        }        if (response.data.code === 401) {  
          useUserStore().logOut();  
          ElMessage.error("登录过期,请重新登录");  
          return;  
        }        return response.data;  
      },      (error: PureHttpError) => {  
        const $error = error;  
        $error.isCancelRequest = Axios.isCancel($error);  
        // 关闭进度条动画  
        NProgress.done();  
        // 所有的响应异常 区分来源为取消请求/非取消请求  
        return Promise.reject($error);  
      }    );  }  /** 通用请求工具函数 */  
  public request<T>(  
    method: RequestMethods,  
    url: string,  
    param?: AxiosRequestConfig,  
    axiosConfig?: PureHttpRequestConfig  
  ): Promise<CommonResult<T>> {  
    const config = {  
      method,  
      url,  
      ...param,  
      ...axiosConfig  
    } as PureHttpRequestConfig;  
  
    // 单独处理自定义请求/响应回调  
    return new Promise((resolve, reject) => {  
      PureHttp.axiosInstance  
        .request(config)  
        .then((response: undefined) => {  
          resolve(response);  
        })        .catch(error => {  
          reject(error);  
        });    });  }  /** 单独抽离的`post`工具函数 */  
  public post<T, P>(  
    url: string,  
    params?: AxiosRequestConfig<P>,  
    config?: PureHttpRequestConfig  
  ): Promise<CommonResult<T>> {  
    return this.request<T>("post", url, params, config);  
  }  
  /** 单独抽离的`get`工具函数 */  
  public get<T, P>(  
    url: string,  
    params?: AxiosRequestConfig<P>,  
    config?: PureHttpRequestConfig  
  ): Promise<CommonResult<T>> {  
    return this.request<T>("get", url, params, config);  
  }}  
  
export const http = new PureHttp();
```


## 头像上传

![[屏幕截图 2024-10-02 215119.jpg]]

上传的头像会经过后端，然后上传至七牛云，数据库则只需要存储七牛云的图片地址即可


### 核心代码

```java
package com.jaever.serve.qiniu.config.service;  
  
import com.fasterxml.jackson.databind.ObjectMapper;  
import com.jaever.serve.db.mapper.SystemUserMapper;  
import com.jaever.serve.qiniu.config.properties.QiniuConfigProperties;  
import com.qiniu.http.Response;  
import com.qiniu.storage.DownloadUrl;  
import com.qiniu.storage.UploadManager;  
import com.qiniu.storage.model.DefaultPutRet;  
import com.qiniu.util.Auth;  
import jakarta.annotation.Resource;  
import org.springframework.stereotype.Service;  
  
import java.io.ByteArrayInputStream;  
import java.io.IOException;  
  
/**  
 * @author gc  
 * @since 2024/9/30  
 */@Service  
public class FileServiceImpl implements FileService{  
    @Resource  
    private QiniuConfigProperties qiniuConfigProperties;  
  
    @Resource  
    private com.qiniu.storage.Configuration qiniuConfig;  
  
    @Resource  
    private ObjectMapper objectMapper;  
  
    @Resource  
    private Auth auth;  
  
    @Resource  
    private SystemUserMapper systemUserMapper;  
  
  
  
    public void upload(byte[] bytes,String key,Integer userId) throws IOException {  
        ByteArrayInputStream inputStream = new ByteArrayInputStream(bytes);  
        // 创建上传token  
        String upToken = auth.uploadToken(qiniuConfigProperties.getBucket());  
        // 创建上传管理器  
        UploadManager uploadManager = new UploadManager(qiniuConfig);  
        // 上传文件  
        Response response = uploadManager.put(inputStream, key, upToken, null, null);  
  
        DefaultPutRet defaultPutRet = objectMapper.readValue(response.bodyString(), DefaultPutRet.class);  
        String url = new DownloadUrl(qiniuConfigProperties.getDomain(), false, defaultPutRet.key).buildURL();  
        systemUserMapper.updateAvatarByStudentUserId(userId,url);  
  
    }}
```

## 课程信息

![[屏幕截图 2024-10-02 215742.jpg]]

课程信息采用树形构建

### 核心代码

```java
package com.jaever.serve.service.campus.course;  
  
import com.jaever.serve.common.errorcode.enums.HttpCodeConstantsEnum;  
import com.jaever.serve.common.exception.BusinessException;  
import com.jaever.serve.common.type.CommonType;  
import com.jaever.serve.controller.admin.campus.course.form.*;  
import com.jaever.serve.controller.admin.campus.course.vo.CourseVO;  
import com.jaever.serve.controller.admin.campus.course.vo.SemesterOptionVO;  
import com.jaever.serve.controller.admin.campus.course.vo.SemesterVO;  
import com.jaever.serve.db.entity.SystemCourseDO;  
import com.jaever.serve.db.mapper.SystemCourseMapper;  
import com.jaever.serve.db.mapper.SystemSemesterMapper;  
import com.jaever.serve.service.campus.course.convert.CourseConvert;  
import com.mybatisflex.core.paginate.Page;  
import com.mybatisflex.core.row.Db;  
import com.mybatisflex.core.update.UpdateChain;  
import jakarta.annotation.Resource;  
import org.springframework.stereotype.Service;  
  
import java.time.LocalDate;  
import java.util.List;  
  
import static com.jaever.serve.db.entity.table.SystemCourseTableDef.system_course;  
  
/**  
 * @author gc  
 * @since 2024/7/24  
 */@Service  
public class CourseServiceImpl implements CourseService {  
  
    @Resource  
    private SystemSemesterMapper systemSemesterMapper;  
  
    @Resource  
    private SystemCourseMapper systemCourseMapper;  
  
    @Override  
    public List<SemesterOptionVO> getSemesterOptionList(){  
        return systemSemesterMapper.selectSemesterOptionList();  
    }  
    @Override  
    public Page<SemesterVO> getSemesterList(SemesterForm semesterForm) {  
        checkCourseTime(semesterForm.getStartTime(),semesterForm.getEndTime());  
        return systemSemesterMapper.paginateAs(  
                semesterForm.getPageNumber(),  
                semesterForm.getPageSize(),  
                systemSemesterMapper.selectCourseList(semesterForm),  
                SemesterVO.class  
        );  
    }  
    @Override  
    public List<CourseVO> getCourseVOList() {  
        List<SystemCourseDO> systemCourseDOList = systemCourseMapper.selectAllWithRelations();  
        return CourseConvert.INSTANCE.convert(systemCourseDOList);  
    }  
    @Override  
    public void addSemester(SemesterAddForm semesterAddForm) {  
        checkCourseTime(semesterAddForm.getStartTime(),semesterAddForm.getEndTime());  
        systemSemesterMapper.insert(CourseConvert.INSTANCE.convert(semesterAddForm));  
    }  
    @Override  
    public void updateSemester(SemesterUpdateForm semesterUpdateForm) {  
        checkCourseTime(semesterUpdateForm.getStartTime(),semesterUpdateForm.getEndTime());  
        systemSemesterMapper.update(CourseConvert.INSTANCE.convert(semesterUpdateForm));  
    }  
    @Override  
    public void deleteSemesterById(Integer id) {  
        systemSemesterMapper.deleteById(id);  
    }  
    @Override  
    public void addCourse(CourseAddForm courseAddForm) {  
        systemCourseMapper.insert(CourseConvert.INSTANCE.convert(courseAddForm));  
    }  
    @Override  
    public void updateCourse(CourseUpdateForm courseUpdateForm) {  
        systemCourseMapper.update(CourseConvert.INSTANCE.convert(courseUpdateForm));  
    }  
    @Override  
    public void deleteCourse(CourseDeleteForm courseDeleteForm) {  
        if (CommonType.Course.COURSE.getCode().equals(courseDeleteForm.getType())) {  
            systemCourseMapper.deleteById(courseDeleteForm.getId());  
        } else {  
            Integer count = systemCourseMapper.getChildCount(courseDeleteForm.getId());  
            if (0 == count) {  
                systemCourseMapper.deleteById(courseDeleteForm.getId());  
            } else {  
                throw new BusinessException(HttpCodeConstantsEnum.INVALID_PARAMETER, "该目录不为空，不能删除");  
            }        }    }  
    @Override  
    public void changeStatus(CourseChangeStatusForm courseChangeStatusForm) {  
            //检查父级节点是否停用，若停用则不允许开启子节点  
            Integer type = systemCourseMapper.checkParentStatus(courseChangeStatusForm.getIdList().getFirst());  
            if (CommonType.DISABLE.getCode().equals(type)){  
                throw new BusinessException(HttpCodeConstantsEnum.INVALID_PARAMETER,"父级节点已停用，不能开启子节点");  
            }        //停止或开启课程,若有子节点则一并改变  
        Db.executeBatch(  
                courseChangeStatusForm.getIdList().size(),  
                1000,  
                SystemCourseMapper.class,  
                (mapper,account)->{  
                    UpdateChain.of(mapper)  
                            .from(system_course)  
                            .set(system_course.status, courseChangeStatusForm.getStatus() )  
                            .where(system_course.id.eq(courseChangeStatusForm.getIdList().get(account)))  
                            .update();  
                }        );    }  
    @Override  
    public void changeType(CourseChangeTypeForm courseChangeTypeForm) {  
        systemCourseMapper.updateTypeById(courseChangeTypeForm.getId(),courseChangeTypeForm.getType());  
    }  
    public void  checkCourseTime(LocalDate startTime, LocalDate endTime){  
        if (startTime != null && endTime != null){  
            if (startTime.isAfter(endTime)){  
                throw new BusinessException(HttpCodeConstantsEnum.INVALID_PARAMETER,"开始时间不能大于结束时间");  
            }        }    }  
  
}
```


```ts
import editForm from "../form.vue";  
import updateEditForm from "../updateForm.vue";  
import { handleTree } from "@/utils/tree";  
import { message } from "@/utils/message";  
import { usePublicHooks } from "@/views/utils/hooks";  
import { addDialog } from "@/components/ReDialog";  
import { reactive, ref, onMounted, h } from "vue";  
import type { FormItemProps, UpdateFormItemProps } from "../utils/types";  
import { cloneDeep, isAllEmpty, deviceDetection } from "@pureadmin/utils";  
import {  
  addCourse,  
  changeStatus,  
  changeType,  
  deleteCourse,  
  getCourseList,  
  getSemesterOptionList,  
  type SemesterOption,  
  updateCourse  
} from "@/api/admin/course";  
import { ElMessage, ElMessageBox } from "element-plus";  
  
export function useCourse() {  
  const form = reactive({  
    courseName: "",  
    courseCode: "",  
    status: null  
  });  
  
  const formRef = ref();  
  const updateFormRef = ref();  
  const dataList = ref([]);  
  const treeData = ref([]);  
  const loading = ref(true);  
  const switchLoadStatusMap = ref({});  
  const switchLoadTypeMap = ref({});  
  const { switchStyle, switchTypeStyle } = usePublicHooks();  
  const semesterOptionList = ref<SemesterOption[]>([]);  
  const columns: TableColumnList = [  
    {      label: "课程ID",  
      prop: "id",  
      width: 180,  
      hide: true  
    },  
    {      label: "课程名称",  
      prop: "courseName",  
      width: 180  
    },  
    {      label: "课程代码",  
      prop: "courseCode",  
      width: 180  
    },  
    {      label: "备注",  
      prop: "remark",  
      width: 180  
    },  
    {      label: "学期",  
      prop: "semester",  
      width: 180  
    },  
    {      label: "学期ID",  
      prop: "semesterId",  
      width: 180,  
      hide: true  
    },  
    {      label: "开始时间",  
      prop: "startTime",  
      width: 180  
    },  
    {      label: "结束时间",  
      prop: "endTime",  
      width: 180  
    },  
    {      label: "排序",  
      prop: "sort",  
      minWidth: 70  
    },  
    {      label: "状态",  
      cellRenderer: scope => (  
        <el-switch  
          size={scope.props.size === "small" ? "small" : "default"}  
          loading={switchLoadStatusMap.value[scope.index]?.loading}  
          v-model={scope.row.status}  
          active-value={1}  
          inactive-value={0}  
          active-text="已启用"  
          inactive-text="已停用"  
          inline-prompt  
          style={switchStyle.value}  
          onChange={() => onChangeStatus(scope as any)}  
        />      ),      minWidth: 90  
    },  
    {      label: "类别",  
      cellRenderer: scope => (  
        <el-switch  
          size={scope.props.size === "small" ? "small" : "default"}  
          loading={switchLoadTypeMap.value[scope.index]?.loading}  
          v-model={scope.row.type}  
          active-value={0}  
          inactive-value={1}  
          active-text="课程大类"  
          inactive-text="具体课程"  
          inline-prompt  
          style={switchTypeStyle.value}  
          onChange={() => onChangeType(scope as any)}  
        />      ),      minWidth: 90  
    },  
    {      label: "操作",  
      fixed: "right",  
      width: 210,  
      slot: "operation"  
    }  
  ];  
  function getAllChildIds(data: any[], id: number): number[] {  
    let childIds: number[] = [id];  
    const findChildren = (currentId: number) => {  
      for (const item of data) {  
        if (item.parentId === currentId) {  
          childIds.push(item.id);  
          findChildren(item.id); // 递归查找子节点  
        }  
      }    };    findChildren(id);  
    return childIds;  
  }  function onChangeStatus({ row, index }) {  
    ElMessageBox.confirm(  
      `确认要<strong>${  
        row.status === 0 ? "停用" : "启用"  
      }</strong><strong style='color:var(--el-color-primary)'>${  
        row.courseName  
      }</strong>吗?`,  
      "系统提示",  
      {        confirmButtonText: "确定",  
        cancelButtonText: "取消",  
        type: "warning",  
        dangerouslyUseHTMLString: true,  
        draggable: true  
      }  
    )      .then(() => {  
        switchLoadStatusMap.value[index] = Object.assign(  
          {},          switchLoadStatusMap.value[index],  
          {            loading: true  
          }  
        );        changeStatus({  
          idList: getAllChildIds(treeData.value, row.id),  
          status: row.status  
        }).then(res => {  
          if (res.success) {  
            message(  
              `已${row.status === 0 ? "停用" : "启用"}${row.courseName}`,  
              {                type: "success"  
              }  
            );          } else {  
            ElMessage.error(res.message);  
          }          onSearch();  
        });        setTimeout(() => {  
          switchLoadStatusMap.value[index] = Object.assign(  
            {},            switchLoadStatusMap.value[index],  
            {              loading: false  
            }  
          );        }, 300);  
      })      .catch(() => {  
        row.status === 0 ? (row.status = 1) : (row.status = 0);  
      });  }  
  function onChangeType({ row, index }) {  
    ElMessageBox.confirm(  
      `确认要修改类别为<strong>${row.type === 0 ? "课程大类" : "具体课程"}`,  
      "系统提示",  
      {        confirmButtonText: "确定",  
        cancelButtonText: "取消",  
        type: "warning",  
        dangerouslyUseHTMLString: true,  
        draggable: true  
      }  
    )      .then(() => {  
        switchLoadTypeMap.value[index] = Object.assign(  
          {},          switchLoadTypeMap.value[index],  
          {            loading: true  
          }  
        );        changeType({  
          id: row.id,  
          type: row.type  
        }).then(res => {  
          if (res.success) {  
            message(`修改成功`, {  
              type: "success"  
            });  
          } else {  
            ElMessage.error(res.message);  
          }          onSearch();  
        });        setTimeout(() => {  
          switchLoadTypeMap.value[index] = Object.assign(  
            {},            switchLoadTypeMap.value[index],  
            {              loading: false  
            }  
          );        }, 300);  
      })      .catch(() => {  
        row.type === 0 ? (row.type = 1) : (row.type = 0);  
      });  }  
  function handleSelectionChange(val) {  
    console.log("handleSelectionChange", val);  
  }  
  function resetForm(formEl) {  
    if (!formEl) return;  
    formEl.resetFields();  
    onSearch();  
  }  
  async function onSearch() {  
    loading.value = true;  
    const { data } = await getCourseList(); // 这里是返回一维数组结构，前端自行处理成树结构，返回格式要求：唯一id加父节点parentId，parentId取父节点id  
    treeData.value = data;  
    let newData = data;  
    // 根据 courseCode 字段进行排序  
    newData.sort((a, b) => (a.sort < b.sort ? -1 : a.sort > b.sort ? 1 : 0));  
    if (!isAllEmpty(form.courseName)) {  
      newData = newData.filter(item =>  
        item.courseName.includes(form.courseName)  
      );    }    if (!isAllEmpty(form.courseCode)) {  
      newData = newData.filter(  
        item =>  
          item.courseCode !== null && item.courseCode.includes(form.courseCode)  
      );    }    if (!isAllEmpty(form.status)) {  
      newData = newData.filter(item => item.status === form.status);  
    }    dataList.value = handleTree(newData); // 处理成树结构  
    setTimeout(() => {  
      loading.value = false;  
    }, 500);  
  }  
  function formatHigherDeptOptions(treeList) {  
    if (!treeList || !treeList.length) return;  
    const newTreeList = [];  
    for (let i = 0; i < treeList.length; i++) {  
      treeList[i].disabled = treeList[i].status === 0 ? true : false;  
      formatHigherDeptOptions(treeList[i].children);  
      newTreeList.push(treeList[i]);  
    }    return newTreeList;  
  }  
  function openDialog(row?: FormItemProps) {  
    addDialog({  
      title: `新增课程`,  
      props: {  
        formInline: {  
          higherDeptOptions: formatHigherDeptOptions(cloneDeep(dataList.value)),  
          id: row?.id ?? null,  
          parentId: row?.parentId ?? null,  
          courseName: row?.courseName ?? "",  
          courseCode: row?.courseCode ?? "",  
          semester: row?.semester ?? "",  
          semesterId: row?.semesterId ?? null,  
          sort: row?.sort ?? 0,  
          status: row?.status ?? 1,  
          remark: row?.remark ?? "",  
          type: row?.type ?? 0  
        },  
        semesterOptionList: semesterOptionList.value  
      },  
      width: "40%",  
      draggable: true,  
      fullscreen: deviceDetection(),  
      fullscreenIcon: true,  
      closeOnClickModal: false,  
      contentRenderer: () => h(editForm, { ref: formRef }),  
      beforeSure: (done, { options }) => {  
        const FormRef = formRef.value.getRef();  
        const curData = options.props.formInline as FormItemProps;  
        function chores() {  
          message(`您新增了课程名称为${curData.courseName}的这条数据`, {  
            type: "success"  
          });  
          done(); // 关闭弹框  
          onSearch(); // 刷新表格数据  
        }  
        FormRef.validate(valid => {  
          if (valid) {  
            // 表单规则校验通过  
            addCourse(curData).then(res => {  
              if (res.success) {  
                chores();  
              } else {  
                ElMessage.error(res.message);  
              }            });          }        });      }    });  }  
  function openUpdateDialog(row?: UpdateFormItemProps) {  
    addDialog({  
      title: `修改课程`,  
      props: {  
        updateFormInline: {  
          higherDeptOptions: formatHigherDeptOptions(cloneDeep(dataList.value)),  
          id: row?.id ?? null,  
          parentId: row?.parentId ?? null,  
          courseName: row?.courseName ?? "",  
          courseCode: row?.courseCode ?? "",  
          semester: row?.semester ?? "",  
          semesterId: row?.semesterId ?? null,  
          sort: row?.sort ?? 0,  
          remark: row?.remark ?? ""  
        },  
        semesterOptionList: semesterOptionList.value  
      },  
      width: "40%",  
      draggable: true,  
      fullscreen: deviceDetection(),  
      fullscreenIcon: true,  
      closeOnClickModal: false,  
      contentRenderer: () => h(updateEditForm, { ref: updateFormRef }),  
      beforeSure: (done, { options }) => {  
        const FormRef = updateFormRef.value.getUpdateRef();  
        const curData = options.props.updateFormInline as UpdateFormItemProps;  
        function chores() {  
          message(`您修改了课程名称为${curData.courseName}的这条数据`, {  
            type: "success"  
          });  
          done(); // 关闭弹框  
          onSearch(); // 刷新表格数据  
        }  
        FormRef.validate(valid => {  
          if (valid) {  
            // 表单规则校验通过  
            updateCourse(curData).then(res => {  
              if (res.success) {  
                chores();  
              } else {  
                ElMessage.error(res.message);  
              }            });          }        });      }    });  }  
  function handleDelete(row) {  
    deleteCourse({ id: row.id, type: row.type }).then(res => {  
      if (res.success) {  
        message(`您删除了课程名称为${row.courseName}的这条数据`, {  
          type: "success"  
        });  
        onSearch();  
      } else {  
        ElMessage.error(res.message);  
      }    });  }  
  onMounted(() => {  
    onSearch();  
    getSemesterOptionList().then(res => {  
      semesterOptionList.value = res.data;  
    });  });  
  return {  
    form,  
    loading,  
    columns,  
    dataList,  
    /** 搜索 */  
    onSearch,  
    /** 重置 */  
    resetForm,  
    /** 新增、修改部门 */  
    openDialog,  
    /** 删除部门 */  
    handleDelete,  
    handleSelectionChange,  
    semesterOptionList,  
    openUpdateDialog  
  };  
}
```

## 角色管理


![[屏幕截图 2024-10-02 220206.jpg]]

角色可以分配权限

### 核心代码


```java
@Transactional(rollbackFor = Exception.class)  
public void updateRoleMenu(UpdateRoleMenuForm updateRoleMenuForm) {  
    //先删除后增加  
    systemRoleMenuMapper.deleteByRoleId(updateRoleMenuForm.getRoleId());  
    systemRoleMenuMapper.insertBatch(RoleConvert.INSTANCE.convert(updateRoleMenuForm));  
}
```


## 菜单管理
![[屏幕截图 2024-10-02 220817.jpg]]

![[屏幕截图 2024-10-02 220727.jpg]]

![[屏幕截图 2024-10-02 220802.jpg]]

### 核心代码
```java
package com.jaever.serve.service.menu;  
  
import com.jaever.serve.common.exception.BusinessException;  
import com.jaever.serve.controller.admin.system.menu.form.MenuAddForm;  
import com.jaever.serve.controller.admin.system.menu.form.MenuUpdateForm;  
import com.jaever.serve.controller.menu.vo.SystemMenuVO;  
import com.jaever.serve.db.entity.SystemMenuDO;  
import com.jaever.serve.db.mapper.SystemMenuMapper;  
import com.jaever.serve.db.mapper.SystemRoleMapper;  
import com.jaever.serve.satoken.util.LoginUserUtils;  
import com.jaever.serve.service.menu.convert.MenuConvert;  
import jakarta.annotation.Resource;  
import org.springframework.stereotype.Service;  
  
import java.util.ArrayList;  
import java.util.List;  
import java.util.Map;  
import java.util.Objects;  
import java.util.function.Function;  
import java.util.stream.Collectors;  
  
import static com.jaever.serve.common.errorcode.enums.HttpCodeConstantsEnum.INVALID_PARAMETER;  
  
/**  
 * @author gc  
 * @since 2024/7/3  
 */@Service  
public class SystemMenuServiceImpl implements SystemMenuService{  
  
    @Resource  
    private SystemMenuMapper systemMenuMapper;  
  
    @Resource  
    SystemRoleMapper systemRoleMapper;  
  
    @Override  
    public List<SystemMenuVO> getMenuList() {  
        Integer userId = Objects.requireNonNull(LoginUserUtils.getLoginUser()).getUserId();  
        List<SystemMenuDO> systemMenuDOList = systemMenuMapper.selectMenuListByRoleId(systemRoleMapper.getRoleIdListByUserId(userId));  
        return getMenuVOList(  
                MenuConvert.INSTANCE.convertSystemMenuVOList(systemMenuDOList)  
        );    }  
    @Override  
    public List<SystemMenuDO> getSystemMenuDOList() {  
        return systemMenuMapper.selectAll();  
    }  
    @Override  
    public void addMenu(MenuAddForm menuAddForm) {  
        systemMenuMapper.insert(MenuConvert.INSTANCE.convert(menuAddForm));  
    }  
    @Override  
    public void updateMenu(MenuUpdateForm menuUpdateForm) {  
        systemMenuMapper.update(MenuConvert.INSTANCE.convert(menuUpdateForm));  
    }  
    @Override  
    public void deleteMenu(Integer id) {  
        //查询该id的菜单是否有子菜单,如果有则不能删除  
        Integer count = systemMenuMapper.selectCountById(id);  
        if (count>0){  
            throw new BusinessException(INVALID_PARAMETER,"该菜单下有子菜单，不能删除");  
        }        systemMenuMapper.deleteById(id);  
    }  
    private List<SystemMenuVO> getMenuVOList(List<SystemMenuVO> roleList) {  
        ArrayList<SystemMenuVO>  systemMenuVOList = new ArrayList<>();  
        Map<Integer, SystemMenuVO> collect = roleList.stream().collect(Collectors.toMap(SystemMenuVO::getId, Function.identity()));  
        roleList.forEach(menu->{  
            if (null == menu.getParentId()) {  
                // 如果是根菜单，添加到roots列表  
                systemMenuVOList.add(menu);  
            }else {  
                // 查找父菜单  
                SystemMenuVO parent = collect.get(menu.getParentId());  
                parent.getChildren().add(menu);  
            }        });        return systemMenuVOList;  
    }}
```