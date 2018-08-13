## 一、springMVC的解题思路
### 初始化过程
1. 解析config文件
2. 加载类信息
3. 根据@Service、@Controller、@Component等实例化类，加入容器中,key beanName,value 实例化对象
4. 扫描注释Autowired，为相应字段赋值
5. 扫描@Controller的@RequestMapping，以及下面方法的@RequestMapping，封装拼接的url，method的到一个类中

### 调用
1. DispatcherServlet，doService ->doDispatch -> 根据url获取匹配的handle，取handle中的method—>最后执行->返回对应的ModelAndView

