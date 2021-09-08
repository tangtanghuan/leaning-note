# 1 数据库设计
## 1.1 表之间关系
![在这里插入图片描述](https://img-blog.csdnimg.cn/09b9277825034651929c97488b6d4836.png)
## 1.2 表设计
（1）sys_user

```bash
CREATE TABLE `sys_user` (
  `user_id` bigint NOT NULL AUTO_INCREMENT,
  `create_by` varchar(255) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `updated_time` datetime DEFAULT NULL,
  `updated_by` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `avatar_name` varchar(255) DEFAULT NULL,
  `avatar_path` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `enabled` tinyint(1) NOT NULL,
  `gender` varchar(255) DEFAULT NULL,
  `is_admin` tinyint(1) DEFAULT NULL,
  `nick_name` varchar(255) DEFAULT NULL,
  `phone` varchar(255) DEFAULT NULL,
  `pwd_reset_time` datetime DEFAULT NULL,
  PRIMARY KEY (`user_id`),
) ENGINE=InnoDB AUTO_INCREMENT=1005 DEFAULT CHARSET=utf8;
```
（2）sys_role

```bash
CREATE TABLE `sys_role` (
  `role_id` bigint NOT NULL AUTO_INCREMENT,
  `create_by` varchar(255) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `updated_time` datetime DEFAULT NULL,
  `updated_by` varchar(255) DEFAULT NULL,
  `description` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`role_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1006 DEFAULT CHARSET=utf8;
```
（3）sys_menu

```bash
CREATE TABLE `sys_menu` (
  `menu_id` bigint NOT NULL AUTO_INCREMENT,
  `create_by` varchar(255) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `updated_time` datetime DEFAULT NULL,
  `updated_by` varchar(255) DEFAULT NULL,
  `cache` tinyint(1) DEFAULT '0',
  `component` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `hidden` tinyint(1) DEFAULT '0',
  `i_frame` tinyint(1) DEFAULT NULL,
  `icon` varchar(255) DEFAULT NULL,
  `menu_sort` int DEFAULT NULL,
  `path` varchar(255) DEFAULT NULL,
  `permission` varchar(255) DEFAULT NULL,
  `pid` bigint DEFAULT NULL,
  `sub_count` int DEFAULT NULL,
  `title` varchar(255) DEFAULT NULL,
  `type` int DEFAULT NULL,
  `redirect` varchar(45) DEFAULT '0',
  PRIMARY KEY (`menu_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1034 DEFAULT CHARSET=utf8;
```
（4）sys_roles_menus

```bash
CREATE TABLE `sys_roles_menus` (
  `role_id` bigint NOT NULL,
  `menu_id` bigint NOT NULL,
  PRIMARY KEY (`role_id`,`menu_id`),
  KEY `FKc67smp0fqtqvu676ed5lt5yfg` (`menu_id`),
  CONSTRAINT `FKafispjhd6o7d07berqyod018y` FOREIGN KEY (`role_id`) REFERENCES `sys_role` (`role_id`),
  CONSTRAINT `FKc67smp0fqtqvu676ed5lt5yfg` FOREIGN KEY (`menu_id`) REFERENCES `sys_menu` (`menu_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
（5）sys_users_roles

```bash
CREATE TABLE `sys_users_roles` (
  `user_id` bigint NOT NULL,
  `role_id` bigint NOT NULL,
  PRIMARY KEY (`user_id`,`role_id`),
  KEY `FK467tnin4c6th4qp8ehba4045p` (`role_id`),
  CONSTRAINT `FK467tnin4c6th4qp8ehba4045p` FOREIGN KEY (`role_id`) REFERENCES `sys_role` (`role_id`),
  CONSTRAINT `FKgm8xovt9418lgk4kkp8h1yi6a` FOREIGN KEY (`user_id`) REFERENCES `sys_user` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
# 2 后端代码
## 2.1 menu相关实体类
（1）Menu和MenuDto实体类

```java
@Entity
@Getter
@Setter
@Table(name = "sys_menu")
public class Menu extends BaseEntity{
    private static final long serialVersionUID = 3931978844810419536L;

    @Id
    @Column(name = "menu_id")
    @NotNull(groups = {Update.class})
    @ApiModelProperty(value = "ID", hidden = true)
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @JsonIgnore
    @ManyToMany(mappedBy = "menus")
    @ApiModelProperty(value = "菜单角色")
    private Set<Role> roles;

    @ApiModelProperty(value = "菜单标题")
    private String title;

    @Column(name = "name")
    @ApiModelProperty(value = "菜单组件名称")
    private String componentName;

    @ApiModelProperty(value = "排序")
    private Integer menuSort = 999;

    @ApiModelProperty(value = "组件路径")
    private String component;

    @ApiModelProperty(value = "路由地址")
    private String path;

    @ApiModelProperty(value = "菜单类型，目录、菜单、按钮")
    private Integer type;

    @ApiModelProperty(value = "权限标识")
    private String permission;

    @ApiModelProperty(value = "菜单图标")
    private String icon;

    @Column(columnDefinition = "TINYINT(1) default 0")
    @ApiModelProperty(value = "缓存")
    private Boolean cache;

    @Column(columnDefinition = "TINYINT(1) default 0")
    @ApiModelProperty(value = "是否隐藏")
    private Boolean hidden;

    @Column(columnDefinition = "VARCHAR(45) default 0")
    @ApiModelProperty(value = "面包屑重定向")
    private String redirect;

    @ApiModelProperty(value = "上级菜单")
    private Long pid;

    @ApiModelProperty(value = "子节点数目", hidden = true)
    private Integer subCount = 0;

    @ApiModelProperty(value = "外链菜单")
    private Boolean iFrame;

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Menu menu = (Menu) o;
        return Objects.equals(id, menu.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

```java
@Getter
@Setter
public class MenuDto extends BaseDTO {
	private static final long serialVersionUID = 2004394224217456711L;

	private Long id;

    private List<MenuDto> children;

    private Integer type;

    private String permission;

    private String title;

    private Integer menuSort;

    private String path;

    private String component;

    private Long pid;

    private Integer subCount;

    private Boolean iFrame;

    private Boolean cache;

    private Boolean hidden;

    private String componentName;

    private String icon;

    private String redirect;

    public Boolean getHasChildren() {
        return subCount > 0;
    }

    public Boolean getLeaf() {
        return subCount <= 0;
    }

    public String getLabel() {
        return title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        MenuDto menuDto = (MenuDto) o;
        return Objects.equals(id, menuDto.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```
（2）MenuRouter菜单路由

```java
@Data
public class MenuRouter {

	/**
	 *
	 *               * { * path: '/system', * component: Layout, * redirect:
	 *               '/system/user', * name: 'System', * meta: { title: '系统管理',
	 *               icon: 'system' ,role: ['admin', 'editor'] }, * children: [ * {
	 *               * path: 'user', * name: 'User', * component: () =>
	 *               import('@/views/system/user/index'), * meta: { title: '用户管理',
	 *               icon: 'user' ,role: ['admin', 'editor'] } * }, * *
	 *
	 */
	/**
	 * 访问路径
	 */
	private String path;
	/**
	 * 组件路径
	 */
	private String component;
	/**
	 * 重定向
	 */
	private String redirect;
	/**
	 * 名称
	 */
	private String name;
	private Map<String, Object> meta;
	private Boolean hidden;
	private List<MenuRouter> children;

}
```
## 2.2 controller层
```java
@Api(tags = "系统：菜单管理")
@RestController
@RequiredArgsConstructor
@RequestMapping("/system/menus")
public class MenuController {

	private final MenuService menuService;
    @ApiOperation("动态生成前端路由")
    @GetMapping("/build")
    public ResponseResult buildMenus() {
    	return ResponseResult.e(HttpStatus.OK, menuService.buildMenus());
    }
 }
```
## 2.3 service层
（1）MenuService
```java
@Service
@RequiredArgsConstructor
public class MenuServiceImpl implements MenuService {

	private final MenuRepository menuRepository;
	private final RoleRepository roleRepository;
	private final UserService userService;
	private final MenuMapper menuMapper;
	
	@Override
	public List<MenuRouter> buildMenus() {
		// 获取用户的菜单目录
		List<MenuDto> resources = this.findByUser(SecurityUtils.getCurrentUserId());
		if (CollectionUtils.isEmpty(resources)) {
			throw new ServiceException("用户未绑定角色，或角色未配置权限", -1);
		}
		if (!resources.isEmpty()) {
			// 过滤
			List<MenuDto> tempResource = resources.stream().filter(menu -> menu.getType() == 1)
					.collect(Collectors.toList());
			// 组织成树结构
			List<MenuDto> treeList = new ArrayList<>();
			for (MenuDto menuDto : tempResource) {
				// 添加一级,layout
				if (menuDto != null && -1L == menuDto.getPid().longValue()) {
					treeList.add(menuDto);
				}
			}
			// 给爸爸找儿子
			treeList = this.addChildrenToParrent(treeList, tempResource);
			// 转换成路由树
			List<MenuRouter> routers = this.toRouterTree(treeList);
			return routers;
		}
		return null;
	}

	@Override
	public List<MenuDto> getChildren(List<MenuDto> menuFirstChildren) {
		List<MenuDto> list = new ArrayList<>();
		menuFirstChildren.forEach(child -> {
			List<MenuDto> menuChildren = this.getMenus(child.getId());
			if (child.getHasChildren()) {
				list.addAll(getChildren(menuChildren));
			}
			list.add(child);
		});
		return list;
	}

	/**
	 * 
	 * @description 获取当前登录用户的菜单
	 * @param currentUserId
	 * @return
	 * @author huan.tang
	 * @date Jan 7, 2021 5:49:17 PM
	 */
	private List<MenuDto> findByUser(Long currentUserId) {
		// 如果是管理员
		if (userService.findById(currentUserId).getIsAdmin()) {
			return menuMapper.toDto(menuRepository.findByNotType(2));
		}
//		List<RoleSmallDto> roles = roleService.findByUserId(currentUserId);
		Set<Role> roles = roleRepository.findByUserId(currentUserId);
		// 返回不为按钮的菜单，及type不等于2
		Set<Menu> menus = roles.stream().flatMap(role -> role.getMenus().stream())
		.filter(menu -> StringUtils.isNotBlank(menu.getPermission()))
		.filter(menu -> menu.getType() != 2).collect(Collectors.toSet());
//		Set<Long> roleIds = roles.stream().map(RoleSmallDto::getId).collect(Collectors.toSet());
//		LinkedHashSet<Menu> menus = menuRepository.findByRoleIdsAndTypeNot(roleIds, 2);
		return menus.stream().map(menuMapper::toDto).collect(Collectors.toList());
	}

	/**
	 * 顶级树找儿子
	 * 
	 * @param operationList 顶级树
	 * @param allResource   资源池
	 */
	private List<MenuDto> addChildrenToParrent(List<MenuDto> operationList, List<MenuDto> allResource) {
		// 得到二级
		operationList.forEach(item -> {
			List<MenuDto> children = new ArrayList<>();
			allResource.forEach(r -> {
				if (item != null && r != null) {
					if (item.getId().equals(r.getPid())) {
						children.add(r);
					}
				}

			});
			this.addChildrenToParrent(children, allResource);
			item.setChildren(children);
		});
		return operationList;
	}

	/**
	 * 将树形结构整理为element框架渲染需要的字段
	 * 
	 * @param resourceList
	 * @return
	 */
	private List<MenuRouter> toRouterTree(List<MenuDto> resourceList) {
		List<MenuRouter> res = new ArrayList<>();
		resourceList.forEach(resource -> {
			MenuRouter route = new MenuRouter();
			route.setPath(resource.getPath());
			route.setName(resource.getComponentName());
			route.setRedirect(resource.getRedirect());
			route.setComponent(resource.getComponent());
			Map<String, Object> meta = new HashMap<>(20);
			route.setHidden(resource.getHidden());
			meta.put("title", resource.getTitle());
			meta.put("icon", resource.getIcon());
			meta.put("noCache", !resource.getCache());
			route.setMeta(meta);
			List<MenuRouter> children = new ArrayList<>();
			if (!resource.getChildren().isEmpty()) {
				children = toRouterTree(resource.getChildren());
			}
			route.setChildren(children);
			res.add(route);
		});
		return res;
	}

	/**
	 * 
	 * @description 设置子菜单
	 * @param menu
	 * @return
	 * @author huan.tang
	 * @date Jan 5, 2021 5:37:18 PM
	 */
	private MenuDto setChildren(MenuDto menu) {
		List<MenuDto> menuChildren = this.getMenus(menu.getId());
		menu.setChildren(menuChildren);
		menuChildren.forEach(child -> {
			if (child.getHasChildren()) {
				setChildren(child);
			}
		});
		return menu;
	}
}
```
（2）UserService

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

	private final UserRepository userRepository;
	
	@Override
	public User findById(Long id) {
		Optional<User> optional = userRepository.findById(id);
		if (!optional.isPresent()) {
			throw new EntityNotFoundException(User.class, "id", id + "");
		}
		return optional.get();
	}
}
```
## 2.4 持久层，使用的是Spring Data JPA
（1）通用Repository

```java
/**
 * @description 通用 Repository
 * @author huan.tang
 * @date Dec 15, 2020
 */
public interface BaseRepository<T, ID> extends JpaRepository<T, ID>, JpaSpecificationExecutor<T>{
	
}
```
（2）MenuRepository

```java
public interface MenuRepository extends BaseRepository<Menu, Long> {
	/**
	 * 
	 * @description 根据类型查询菜单
	 * @param i
	 * @return
	 * @author huan.tang
	 * @date Jan 7, 2021 5:54:28 PM
	 */
	@Query(value = "select * from sys_menu where type != ?1", nativeQuery = true)
	List<Menu> findByNotType(int type);
}
```
（3）RoleRepository

```java
public interface RoleRepository extends BaseRepository<Role, Long> {
	/**
	 * 
	 * @description 根据用户id查询role
	 * @return
	 * @author huan.tang
	 * @date Dec 17, 2020 3:07:12 PM
	 */
	@Query(value = "select * from sys_role where role_id in "
			+ "(select role_id from sys_users_roles where user_id = ?1)", nativeQuery = true)
	Set<Role> findByUserId(Long uid);
}
```
# 3 前端代码
（1）在src/router目录下添加懒加载components脚本
 - 开发环境
 _import_development.js

```javascript
/**
 * 后台传回的菜单
 */
module.exports = file => require('@/views/' + file + '.vue').default // vue-loader at least v13.0.0+
```
 * 生产环境
 _import_production.js
 

```javascript
/**
 * 加载后台传来的路由
 */
module.exports = file => () => import('@/views/' + file + '.vue')

```
（2）在src/api目录下添加resource.js，用于请求后台菜单

```javascript
import request from '@/utils/request'

/**
 * 加载用户路由
 */
export function fetchUserRoute() {
  return request({
    url: '/system/menus/build',
    method: 'get'
  })
}
```
（3）修改src/store/modules/permission.js文件

```javascript
import { constantRoutes } from '@/router'
import { fetchUserRoute } from '@/api/system/resource'
import Layout from '@/layout'
const import_router = require('@/router/_import_' + process.env.NODE_ENV)

/**
 * Use meta.role to determine if the current user has permission
 * @param roles
 * @param route
 */
function hasPermission(roles, route) {
  if (route.meta && route.meta.roles) {
    return roles.some(role => route.meta.roles.includes(role))
  } else {
    return true
  }
}

/**
 * Filter asynchronous routing tables by recursion
 * @param routes asyncRoutes
 * @param roles
 */
export function filterAsyncRoutes(routes, roles) {
  const res = []

  routes.forEach(route => {
    const tmp = { ...route }
    if (hasPermission(roles, tmp)) {
      if (tmp.children) {
        tmp.children = filterAsyncRoutes(tmp.children, roles)
      }
      res.push(tmp)
    }
  })
  return res
}

export function filterAsyncRouter(asyncRouterMap) { // 遍历后台传来的路由字符串，转换为组件对象
  const res = []
  asyncRouterMap.forEach(route => {
    if (route.component) {
      if (route.component === '@/layout') { // Layout组件特殊处理
        route.component = Layout
      } else {
        route.component = import_router(route.component)
      }
    }
    if (route.children.length === 0) { // 如果儿子只有一个，删除儿子的空children，不然显示会有箭头
      delete route.children
    }
    if (route.children) {
      // 递归
      route.children = filterAsyncRouter(route.children)
    }
    res.push(route)
  })
  return res
}

const state = {
  routes: [],
  addRoutes: []
}

const mutations = {
  SET_ROUTES: (state, routes) => {
    state.addRoutes = routes
    state.routes = constantRoutes.concat(routes)
  }
}

const actions = {
  // get routes by user role from backend
  generateRoutes({ commit }, roles) {
    // get request
    return new Promise(resolve => {
      // call method
      fetchUserRoute().then(response => {
        const data = response.data
        let accessedRoutes = []
        accessedRoutes = filterAsyncRouter(data)
        commit('SET_ROUTES', accessedRoutes)
        resolve(accessedRoutes)
      }).catch(error => {
        Promise.reject(error)
      })
    })
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}

```
（4）修改src/permission.js文件

```javascript
import router from './router'
import store from './store'
import { Message } from 'element-ui'
import NProgress from 'nprogress' // progress bar
import 'nprogress/nprogress.css' // progress bar style
import { getToken } from '@/utils/auth' // get token from cookie
import getPageTitle from '@/utils/get-page-title'

NProgress.configure({ showSpinner: false }) // NProgress Configuration

const whiteList = ['/login', '/auth-redirect'] // no redirect whitelist

router.beforeEach(async(to, from, next) => {
  // start progress bar
  NProgress.start()

  // set page title
  document.title = getPageTitle(to.meta.title)

  // determine whether the user has logged in
  const hasToken = getToken()

  if (hasToken) {
    if (to.path === '/login') {
      // if is logged in, redirect to the home page
      next({ path: '/' })
      NProgress.done() // hack: https://github.com/PanJiaChen/vue-element-admin/pull/2939
    } else {
      // determine whether the user has obtained his permission roles through getInfo
      const hasRoles = store.getters.roles && store.getters.roles.length > 0
      if (hasRoles) {
        next()
      } else {
        try {
          // get user info
          // note: roles must be a object array! such as: ['admin'] or ,['developer','editor']
          const { roles } = await store.dispatch('user/getInfo')

          // generate accessible routes map based on roles
          const accessRoutes = await store.dispatch('permission/generateRoutes', roles)

          // dynamically add accessible routes
          router.addRoutes(accessRoutes)

          // hack method to ensure that addRoutes is complete
          // set the replace: true, so the navigation will not leave a history record
          next({ ...to, replace: true })
        } catch (error) {
          // remove token and go to login page to re-login
          await store.dispatch('user/resetToken')
          Message.error(error || 'Has Error')
          next(`/login?redirect=${to.path}`)
          NProgress.done()
        }
      }
    }
  } else {
    /* has no token*/

    if (whiteList.indexOf(to.path) !== -1) {
      // in the free login whitelist, go directly
      next()
    } else {
      // other pages that do not have permission to access are redirected to the login page.
      next(`/login?redirect=${to.path}`)
      NProgress.done()
    }
  }
})

router.afterEach(() => {
  // finish progress bar
  NProgress.done()
})

```
（5）src/router/index.js只保留静态路由

```javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

/* Layout */
import Layout from '@/layout'

/**
 * Note: sub-menu only appear when route children.length >= 1
 * Detail see: https://panjiachen.github.io/vue-element-admin-site/guide/essentials/router-and-nav.html
 *
 * hidden: true                   if set true, item will not show in the sidebar(default is false)
 * alwaysShow: true               if set true, will always show the root menu
 *                                if not set alwaysShow, when item has more than one children route,
 *                                it will becomes nested mode, otherwise not show the root menu
 * redirect: noRedirect           if set noRedirect will no redirect in the breadcrumb
 * name:'router-name'             the name is used by <keep-alive> (must set!!!)
 * meta : {
    roles: ['admin','editor']    control the page roles (you can set multiple roles)
    title: 'title'               the name show in sidebar and breadcrumb (recommend set)
    icon: 'svg-name'/'el-icon-x' the icon show in the sidebar
    noCache: true                if set true, the page will no be cached(default is false)
    affix: true                  if set true, the tag will affix in the tags-view
    breadcrumb: false            if set false, the item will hidden in breadcrumb(default is true)
    activeMenu: '/example/list'  if set path, the sidebar will highlight the path you set
  }
 */

/**
 * constantRoutes
 * a base page that does not have permission requirements
 * all roles can be accessed
 */
export const constantRoutes = [
  {
    path: '/redirect',
    component: Layout,
    hidden: true,
    children: [
      {
        path: '/redirect/:path(.*)',
        component: () => import('@/views/redirect/index')
      }
    ]
  },
  {
    path: '/login',
    component: () => import('@/views/login/index'),
    hidden: true
  },
  {
    path: '/auth-redirect',
    component: () => import('@/views/login/auth-redirect'),
    hidden: true
  },
  {
    path: '/404',
    component: () => import('@/views/error-page/404'),
    hidden: true
  },
  {
    path: '/401',
    component: () => import('@/views/error-page/401'),
    hidden: true
  },
  {
    path: '/',
    component: Layout,
    redirect: '/home',
    children: [
      {
        path: 'home',
        component: () => import('@/views/home/index'),
        name: 'Home',
        meta: { title: 'home', icon: 'star', affix: true }
      }
    ]
  },
  {
    path: '/profile',
    component: Layout,
    hidden: true,
    children: [
      {
        path: 'index',
        component: () => import('@/views/profile/index'),
        name: 'profile',
        meta: { title: 'profile', breadcrumb: false }
      }
    ]
  }
  // 404 page must be placed at the end !!!
  // { path: '*', redirect: '/404', hidden: true }
]

const createRouter = () => new Router({
  // mode: 'history', // require service support
  scrollBehavior: () => ({ y: 0 }),
  routes: constantRoutes
})

const router = createRouter()

// Detail see: https://github.com/vuejs/vue-router/issues/1234#issuecomment-357941465
export function resetRouter() {
  const newRouter = createRouter()
  router.matcher = newRouter.matcher // reset router
}

export default router

```

