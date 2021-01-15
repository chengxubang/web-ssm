ssm搭建拍卖系统
=

程序有问题，找[程序帮](http://suo.nz/530ijn)：QQ1022287044


项目介绍及流程
----

>拍卖系统采用ssm架构搭建,页面数据渲染采用jsp的初级简易拍卖系统,拍卖流程为普通用户注册、登录后，进入个人中心发布商品后，系统管理员进行商品审核，审核不通过，用户发布列表展示拒绝原因，审核通过后，首页及分类可查询展示，非发布人员进行商品竞价，竞价完成，发布者进行商品发货，拍卖获得者进行付款，完成最终交易。

开发环境：
-----
1. jdk 8
2. intellij idea
3. tomcat 8
4. mysql 5.7

所用技术：
-----
项目框架：ssm

前端渲染：jsp

前端技术：js/jquery


项目目录结构
-----
![项目结构](/image/项目结构.png)

- controller，视图控制层，
- service，业务逻辑层
- dao，数据库操作层
- jsp，页面进行数据渲染
- mapper.xml,此文件定义了操作数据库的sql，每个sql是一个statement，映射文件是mybatis的核心
- jdbc.properties，数据库帐号密码配置

运行效果
----

- 注册 

 ![注册](/image/注册.png)

- 首页

![首页](/image/首页.png)

- 发布商品

![发布商品](/image/发布商品.png)

- 商品竞价

![商品竞价](/image/商品竞价.png)

- 浏览记录

![浏览记录](/image/浏览记录.png)

- 后端-商品管理

![后端-商品管理](/image/后端-商品管理.png)



核心项目配置:
-----
1.spring-mvc.xml配置：
```diff
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-4.0.xsd
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
    <mvc:annotation-driven/>
    <!-- 自动扫描  @Controller-->
    <context:component-scan base-package="com.pmxt.controller"/>
    <mvc:resources mapping="/js/**" location="/js/"/>
    <mvc:resources mapping="/css/**" location="/css/"/>
    <mvc:resources mapping="/images/**" location="/images/"/>
    <mvc:resources mapping="/view/**" location="/view/"/>
    <mvc:resources mapping="/upload/**" location="/upload/"/>
    <!--避免IE执行AJAX时，返回JSON出现下载文件 -->
    <bean id="mappingJacksonHttpMessageConverter"
          class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
        <property name="supportedMediaTypes">
            <list>
                <value>text/html;charset=UTF-8</value>
            </list>
        </property>
    </bean>
    <!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
        <property name="messageConverters">
            <list>
                <ref bean="mappingJacksonHttpMessageConverter"/> <!-- JSON转换器 -->
            </list>
        </property>
    </bean>


    <!-- 定义跳转的文件的前后缀 ，视图模式配置 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 文件上传配置 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 默认编码 -->
        <property name="defaultEncoding" value="UTF-8"/>
        <!-- 上传文件大小限制为31M，31*1024*1024 -->
        <property name="maxUploadSize" value="32505856"/>
        <!-- 内存中的最大值 -->
        <property name="maxInMemorySize" value="4096"/>
    </bean>


    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <mvc:exclude-mapping path="/showLogin"/>
            <mvc:exclude-mapping path="/getVerifyCode"/>
            <mvc:exclude-mapping path="/checkLogin"/>
            <mvc:exclude-mapping path="/showReg"/>
            <mvc:exclude-mapping path="/forget"/>
            <mvc:exclude-mapping path="/checkUname"/>
            <mvc:exclude-mapping path="/forgetPassword"/>
            <mvc:exclude-mapping path="/reg"/>
            <mvc:exclude-mapping path="/css/**"/>
            <mvc:exclude-mapping path="/images/**"/>
            <mvc:exclude-mapping path="/js/**"/>
            <mvc:exclude-mapping path="/upload/**"/>
            <mvc:exclude-mapping path="/WEB-INF/jsp/login.jsp"/>

            <bean class="com.pmxt.interceptor.LoginInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>

    <!--500错误和404请求失败-->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <map>
                <entry key="ResourceNotFoundException" value="common/error/resourceNotFoundError" />
                <entry key=".DataAccessException" value="common/error/dataAccessError" />
            </map>
        </property>
        <property name="statusCodes">
            <map>
                <entry key="common/error/resourceNotFoundError" value="404" />
                <entry key="common/error/dataAccessError" value="500" />
            </map>
        </property>
    </bean>

</beans>
```

2.spring-mybatis.xml配置
```diff
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd
                        http://www.springframework.org/schema/tx
                        http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 自动扫描 -->
    <context:component-scan base-package="com.pmxt"/>


    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName">
            <value>com.mysql.jdbc.Driver</value>
        </property>
        <property name="url">
            <value>jdbc:mysql://localhost:3306/pmxt?characterEncoding=UTF-8</value>

        </property>
        <property name="username">
            <value>root</value>
        </property>
        <property name="password">
            <value>root123</value>
        </property>
    </bean>

    <!-- mybatis和spring完美整合，不需要mybatis的配置映射文件 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 自动扫描mapping.xml文件 -->
        <property name="mapperLocations" value="classpath:mapping/*.xml"></property>
    </bean>

    <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.pmxt.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
    </bean>


    <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
``` 

3.登录验证
```
@RequestMapping("/checkLogin")
@ResponseBody
public int checkLogin(HttpServletRequest request, String uname, String upassword, String verifyCode) {
    HttpSession session = request.getSession();
    String sessionVerifyCode = (String) session.getAttribute("verifyCodeValue");
    if (!verifyCode.equalsIgnoreCase(sessionVerifyCode)) {
        int flag = 4;
        return flag;

    } else {
        List<PUsersModel> pUsersModels = pUserService.checkLogin(uname, upassword);
        if (pUsersModels.isEmpty()) {
            return 0;//用户名不存在
        }
        String relpassword = pUsersModels.get(0).getUpassword();
        int utype = pUsersModels.get(0).getUtype();
        if (upassword.equals(relpassword)) {
            session = request.getSession();
            session.setAttribute("uname", uname);
            session.setAttribute("utype", utype);
            session.setAttribute("pUsersModel", pUsersModels.get(0));
            if (utype == 1) {
                return 3;//管理员
            }
            return 1;//密码正确
        } else {
            return 2;//密码不正确
        }
    }
}
```

4.系统推荐
```diff
@RequestMapping("/sysRecommend")
public String sysRecommend(HttpSession session, Model model, String page)
{
    PUsersModel pUsersModel=(PUsersModel)session.getAttribute("pUsersModel");
    //先判断当前用户是否已经浏览过商品
    LookshistoryModel lookModel=new LookshistoryModel();
    lookModel.setUserid(pUsersModel.getUid());
    List<LookshistoryModel> lists=lookshistoryService.selectLookshistory(lookModel);		//查询总记录并做降序处理
    Map<Integer,Integer> gmap=new HashMap<Integer, Integer>();
    if(lists!=null&lists.size()>0){ 														//根据做商品过滤筛选 精准推荐
        int hits=0;																			//获取浏览总数
        for(LookshistoryModel m:lists){
            gmap.put(m.getPgtype(),m.getHits());											//填充商品占比情况
            hits+=m.getHits();																//叠加浏览次数
        }
        Map<Integer,Integer> gpmap=new HashMap<Integer, Integer>();										//对象储存转换，类型--->>次数 模式
        for (Map.Entry<Integer, Integer> entry : gmap.entrySet()) {
            BigDecimal percentage=new BigDecimal(entry.getValue()).divide(new BigDecimal(hits),2,BigDecimal.ROUND_HALF_DOWN);		//当前最高商品的占比
            int result=percentage.multiply(new BigDecimal(12)).intValue();							//推荐每个商品出现的占比情况
            gpmap.put(entry.getKey(),result);
        }
        List<GoodsModel> afterGoodsModels =new ArrayList<GoodsModel>();
        int afterPageNums =0;
        for (Map.Entry<Integer, Integer> entry : gmap.entrySet()) {								//最终推荐出商品情况
            int afterPageItem = goodsService.countAfterGoodsByKinds(entry.getKey());
            final int afterPageSize = entry.getValue();
            if (page == null)
            {
                page = "1";
            }
            PageUtil afterPageUtil = new PageUtil(afterPageItem, afterPageSize, Integer.parseInt(page));
            int afterPageNum = afterPageUtil.getPageNum();
            afterPageNums+=afterPageNum;
            int afterStartRow = afterPageUtil.getStartRow();
            List<GoodsModel> goodsModels = goodsService.getAfterCheckGoodsByPageBykinds(afterStartRow, afterPageSize, entry.getKey());
            Boolean afterHasNextPage = (Integer.parseInt(page) != afterPageNum) && (afterPageNum != 0);
            model.addAttribute("afterHasPrePage", Integer.parseInt(page) != 1);
            model.addAttribute("afterHasNextPage", afterHasNextPage);
            model.addAttribute("afterGoodsModels", afterGoodsModels);
            model.addAttribute("afterPageItem", afterPageItem);

            model.addAttribute("nowPage", Integer.parseInt(page));
            afterGoodsModels.addAll(goodsModels);
        }
        model.addAttribute("afterPrePage", Integer.parseInt(page) - 1);
        model.addAttribute("afterNextPage", Integer.parseInt(page) + 1);
        model.addAttribute("afterPageNum", afterPageNums);		//总页面
        model.addAttribute("afterGoodsModels", afterGoodsModels);//总商品

    }else{																					//根据大众浏览推荐

        lists=lookshistoryService.selectLookshistory(new LookshistoryModel());				//根据大众浏览次数多的多推荐
        if(lists!=null&lists.size()>0){
            int hits=0;																			//获取浏览总数
            for(LookshistoryModel m:lists){
                gmap.put(m.getPgtype(),m.getHits());											//填充商品占比情况
                hits+=m.getHits();																//叠加浏览次数
            }
            Map<Integer,Integer> gpmap=new HashMap<Integer, Integer>();//商品占比存储对象
            for (Map.Entry<Integer, Integer> entry : gmap.entrySet()) {
                BigDecimal percentage=new BigDecimal(entry.getValue()).divide(new BigDecimal(hits));		//当前最高商品的占比
                int result=percentage.multiply(new BigDecimal(12)).intValue();							//推荐每个商品出现的占比情况
                gpmap.put(entry.getKey(),result);
            }
            List<GoodsModel> afterGoodsModels =new ArrayList<GoodsModel>();
            int afterPageNums =0;
            for (Map.Entry<Integer, Integer> entry : gmap.entrySet()) {				//最终推荐出商品情况
                int afterPageItem = goodsService.countAfterGoodsByKinds(entry.getKey());
                final int afterPageSize = entry.getValue();
                if (page == null)
                {
                    page = "1";
                }``
                PageUtil afterPageUtil = new PageUtil(afterPageItem, afterPageSize, Integer.parseInt(page));
                int afterPageNum = afterPageUtil.getPageNum();
                afterPageNums+=afterPageNum;
                int afterStartRow = afterPageUtil.getStartRow();
                List<GoodsModel> goodsModels = goodsService.getAfterCheckGoodsByPageBykinds(afterStartRow, afterPageSize, entry.getKey());
                Boolean afterHasNextPage = (Integer.parseInt(page) != afterPageNum) && (afterPageNum != 0);
                model.addAttribute("afterHasPrePage", Integer.parseInt(page) != 1);
                model.addAttribute("afterHasNextPage", afterHasNextPage);
                model.addAttribute("afterGoodsModels", afterGoodsModels);
                model.addAttribute("afterPageItem", afterPageItem);

                model.addAttribute("nowPage", Integer.parseInt(page));
                afterGoodsModels.addAll(goodsModels);
            }
            model.addAttribute("afterPrePage", Integer.parseInt(page) - 1);
            model.addAttribute("afterNextPage", Integer.parseInt(page) + 1);
            model.addAttribute("afterPageNum", afterPageNums);		//总页面
            model.addAttribute("afterGoodsModels", afterGoodsModels);//总商品
        }else{
            int lindid=(int)Math.random()*(1);									//没有浏览次数系统随机推荐
            int afterPageItem = goodsService.countAfterGoodsByKinds(lindid);
            final int afterPageSize = 12;
            if (page == null)
            {
                page = "1";
            }
            PageUtil afterPageUtil = new PageUtil(afterPageItem, afterPageSize, Integer.parseInt(page));
            int afterPageNum = afterPageUtil.getPageNum();
            int afterStartRow = afterPageUtil.getStartRow();
            List<GoodsModel> afterGoodsModels = goodsService.getAfterCheckGoodsByPageBykinds(afterStartRow, afterPageSize, 0);
            Boolean afterHasNextPage = (Integer.parseInt(page) != afterPageNum) && (afterPageNum != 0);
            model.addAttribute("afterHasPrePage", Integer.parseInt(page) != 1);
            model.addAttribute("afterHasNextPage", afterHasNextPage);
            model.addAttribute("afterPrePage", Integer.parseInt(page) - 1);
            model.addAttribute("afterNextPage", Integer.parseInt(page) + 1);
            model.addAttribute("afterGoodsModels", afterGoodsModels);
            model.addAttribute("afterPageItem", afterPageItem);
            model.addAttribute("afterPageNum", afterPageNum);
            model.addAttribute("nowPage", Integer.parseInt(page));
        }
    }
    return "otherkinds";
}

```



项目总结
----

目前此版本为简易拍卖系统，其他拍卖逻辑或者额外功能，后续迭代排期开发，敬请关注！

