xsd ---->定义了schema标签的namespace
wsdl ---->定义了wsdl标签的namespace

<definitions>

	<types>
		<schema>
			<element>
		</schema>
	</types>
	<message>
	</message>
	<portType>
	</portType>
	
	<binding>
	</binding>
	<services>
		<port>
		</port>
	</services>
</definitions>







### 课程安排：

	web service 2
	ibatis (mybatis) 1

#### 复习2个相关的重要知识

	a schema约束
		1. namespace : schema文件的标识属性，相当于id,每个schema文件需要有一个唯一的namespace值
		2. targetNameSpace ：指定当前schema文件的namespace值
		3. xmlns ：引入一个schema约束，它的值为一个schema的namespace值
		4. element ：定义标签
	b. Http协议
		1. 请求的组成：
			请求行，请求头，请求体(Post)
			client--》server
		2. 响应的组成
			状态行，响应头，响应体
			server-->client
		3. 请求的过程

### webservice

	http+xml(schema)
#### Web service是什么？

	web服务：服务器端整出一些资源可以让客户端应用访问（获取数据）
	一个跨语言、跨平台的规范（抽象）
	多个跨平台、跨语言的应用间通信整合的方案（实际）

#### 为什么要用Web service？

	业务需求：
		应用A: java写的,运行在windows平台下
			List<User> getAllUsers();
		应用B: c语言写的，运行在linux平台下
			需要请求应用A的getAllUsers()得到所有的user信息来展示
	web service能解决：
		跨平台调用 
		跨语言调用
		远程调用

#### 什么时候使用web Service?

	同一家公司的新旧应用
	不同公司的应用间
		分析业务需求：淘宝网与顺风物流系统如何交互？
	一些提供数据的内容聚合应用：天气预报、股票行情

#### 如何做web service的开发?

	服务器端（处理客户端应用的请求,执行业务逻辑，提供数据）
	客户端（发送请求，获取数据）

##### 几个常用的

```
WSDL：web service definition language
	对应一种类型的文件.wsdl
	一个web service对应一个唯一的wsdl文档
	定义了web service的服务器端与客户端应用交互传递请求和响应数据的格式和方式
SOAP：simple object access protocal
	http+xml片断
	soap消息：请求消息和响应消息
	它依赖于wsdl文档的定义
SEI：Service EndPoint Interface
	web service的终端接口，就是服务器端用来处理请求的接口（其中的方法就是处理请求的方法）
CXF：Celtix and XFire
	一个apache的webservice框架
```

##### 开发web service

	1. 使用JDK开发
###### ①. 服务器端

	编码：
		a. 创建一个基于jdk6以上版本的java工程
		b. 定义SEI web service Endpoint interface(web service终端接口)
			@WebService
			public interface HelloWS {
				@WebMethod
				public String sayHello(String name);
			}
		c. 定义SEI的实现类：
			@WebService
			public class HelloWSImpl implements HelloWS {
				@Override
				public String sayHello(String name) {
					System.out.println("sayHello "+name);
					return "hello "+name;
				}
			}
	
	发布：
		public class Server {
			public static void main(String[] args) {
				//客户端发送web service请求的url
				String address = "http://192.168.1.104/ws_server/helloWS";
				//处理请求的SEI对象
				HelloWS helloWS = new HelloWSImpl();
				//发布web service
				Endpoint.publish(address, helloWS);
				System.out.println("发布web service成功！");
			}
		}
###### ②. 客户端

	1. eclipse Web Service浏览器
		a. 查看Web Service所对应的WSDL文档：...?wsdl
		b. 使用eclipse访问
			请求体：SOAP Request Envelope
				<soapenv:Envelope>
					<soapenv:Body>
						<q0:sayHello>
							<arg0>tt</arg0>
						</q0:sayHello>
					</soapenv:Body>
				</soapenv:Envelope>
			响应体：SOAP Response Envelope
				<S:Envelope>
					<S:Body>
						<ns2:sayHelloResponse xmlns:ns2="http://ws.java.atguigu.net/">
							<return>hello tt</return> 
						</ns2:sayHelloResponse>
					</S:Body>
				 </S:Envelope>
	2. 编码实现
		a. 创建客户端java应用
		b. 在应用的src下执行cxf的命令生成客户端代码：
			wsimport -keep http://192.168.1.104/ws_server/helloWS?wsdl
		c. 编写客户端调用的测试代码，执行：
			public class Client {
				public static void main(String[] args) {
					//创建SEI的工厂对象
					HelloWSImplService factory = new HelloWSImplService();
					//得到一个SEI实现类对象
					HelloWSImpl helloWS = factory.getHelloWSImplPort();
					//调用SEI的方法，此时才去发送web Service请求，并得到返回结果
					String result = helloWS.sayHello("Tom");
					System.out.println(result);
				}
			}
					
		③.请求过程记录：使用eclipse的tcp/ip工具进行请求的监控
1. 配置tcp/ip监视器（请求转发+请求信息记录）
 监听port : 80(wsdl文件中的address属性一致)
 监听主机 ：ip
 转发的port : 8989(server端一致)
2. 将webservice的wsdl文件保存到client应用中: helloWS.wsdl
3. 修改helloWS.wsdl文件中的uri : port adresss属性 8989-->80
4. 在client应用的src下执行命令: wsimport -keep 本地wsdl文件---》生成client端代码
5. 借助生成的代码请求web service
   ​		
④.应用：编写天气预报web service的客户端(根据城市名称得到城市今天的天气信息)
	1. 找到对应的wsdl文件所在的网络地址：http://webservice.webxml.com.cn/WebServices/WeatherWebService.asmx?wsdl
	2. 通过eclipse的web service浏览请求，测试是否可用
	3. 将wsdl文档保存到本地：ws_weath应用文件夹下weath.wsdl
		(将<s:element ref="s:schema" /><s:any /> 替换成 <s:any minOccurs="2" maxOccurs="2"/>)
	4. ws_weath应用下执行命令：wsimport -keep weath.wsdl的路径  ---》生成对应的客户端代码
	5. 编写测试程序：
	public static void main(String[] args) {
		WeatherWebService factory = new WeatherWebService();
		WeatherWebServiceSoap webServiceSoap = factory.getWeatherWebServiceSoap();
		ArrayOfString arrayOfString = webServiceSoap.getWeatherbyCityName("荆州");
		List<String> list = arrayOfString.getString();
		for(String s : list) {
			System.out.println(s);
		}
	}
	
	2. 使用CXF框架开发
	
		①.CXF : xfire-->xfire + celtrix
			做web service开发的开源框架
			
		②.开发Server端：
			加入cxf的Jar包即可，其它不需要动
			
		③.wsdl文件分析 ：
			<?xml version='1.0' encoding='UTF-8'?>
			<!-- 
				wsdl的作用：定义了web service的服务器端与客户端应用交互传递请求和响应数据的格式和方式
					请求的url
			-->
			<wsdl:definitions 
				xmlns:xsd="http://www.w3.org/2001/XMLSchema"
				xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/" 
				xmlns:tns="http://ws.java.atguigu.net/"
				xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" 
				xmlns:ns1="http://schemas.xmlsoap.org/soap/http"
				name="HelloWSImplService" 
				targetNamespace="http://ws.java.atguigu.net/">
				<!-- 
					定义当前wsdl文档中所使用的标签，很重要的一部分为soap消息中xml片断所包含的一些标签
					<sayHello>
						<arg0>文本</arg0>
					</sayHello>
						===>soap请求的请求体的一部分
					<sayHelloResponse>
						<return></return>
					</sayHelloResponse>
						===>soap响应的请求体的一部分
				 -->
				<wsdl:types>
					<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
						xmlns:tns="http://ws.java.atguigu.net/" 
						elementFormDefault="unqualified"
						targetNamespace="http://ws.java.atguigu.net/" version="1.0">
						
						<xs:element name="sayHello" type="tns:sayHello" />
						<xs:element name="sayHelloResponse" type="tns:sayHelloResponse" />
						
						<xs:complexType name="sayHello">
							<xs:sequence>
								<xs:element minOccurs="0" name="arg0" type="xs:string" />
							</xs:sequence>
						</xs:complexType>
						<xs:complexType name="sayHelloResponse">
							<xs:sequence>
								<xs:element minOccurs="0" name="return" type="xs:string" />
							</xs:sequence>
						</xs:complexType>
					</xs:schema>
				</wsdl:types>
				
				<!-- 
					定义请求消息，它的个数为SEI方法个数的2倍
						message name 指定消息的名称
							part element 指定消息的组成，依赖<types>中定义的某个element 
				 -->
				<wsdl:message name="sayHelloResponse">
					<wsdl:part element="tns:sayHelloResponse" name="parameters">
					</wsdl:part>
				</wsdl:message>
				<wsdl:message name="sayHello">
					<wsdl:part element="tns:sayHello" name="parameters">
					</wsdl:part>
				</wsdl:message>
				
				<!-- 
					定义SEI接口
						portType name 指定SEI接口名
							operation 接口的操作，也就是其方法
								input message 服务器端接收的消息,依赖指定的<message>,也就是将哪个消息作为请求消息对应方法的参数
								output message 指定服务器返回的消息，依赖指定的<message>，也就是将哪个消息作为响应消息对应方法的返回值
				 -->
				<wsdl:portType name="HelloWS">
					<wsdl:operation name="sayHello">
						<wsdl:input message="tns:sayHello" name="sayHello">
						</wsdl:input>
						<wsdl:output message="tns:sayHelloResponse" name="sayHelloResponse">
						</wsdl:output>
					</wsdl:operation>
				</wsdl:portType>
				<!-- 
					定义服务器端处理请求的SEI实现类对象
						binding type ：参照<portType>所定义的SEI
							soap:binding : 绑定的方式，也就数据传递的方式 为文档（即xml）
								input : 指定请求消息（与<portType>中的input对应）
								output：指定响应消息（与<portType>中的output对应）
									body use="literal" 消息体为文本
				 -->
				<wsdl:binding name="HelloWSImplServiceSoapBinding" type="tns:HelloWS">
					<soap:binding style="document"
						transport="http://schemas.xmlsoap.org/soap/http" />
					<wsdl:operation name="sayHello">
						<soap:operation soapAction="" style="document" />
						<wsdl:input name="sayHello">
							<soap:body use="literal" />
						</wsdl:input>
						<wsdl:output name="sayHelloResponse">
							<soap:body use="literal" />
						</wsdl:output>
					</wsdl:operation>
				</wsdl:binding>
				<!-- 定义客户端调用web service的入口
						service name ：生产客户端的SEI接口实现的工厂类
							port binding ：处理请求的服务器端的SEI接口实现类对象
								address location ：web service的请求url	
				-->
				<wsdl:service name="HelloWSImplService">
					<wsdl:port binding="tns:HelloWSImplServiceSoapBinding" name="HelloWSImplPort">
						<soap:address location="http://192.168.1.104/ws_server/helloWS" />
					</wsdl:port>
				</wsdl:service>
			</wsdl:definitions>
		
		④.web service 处理复杂数据
			1. 数组集合（List,Set,Map）
			2. 自定义类型

⑤. 一次web service请求的流程(以请求getStudentById(int id)它为例)
​	
1. 客户端调用：helloWS.getStudentById(1)，根据wsdl文档生成一个xml片断
 <?xml version="1.0"  standalone="no"?>
 <S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
 	<S:Body>
 		<ns2:getStudentById xmlns:ns2="http://server.ws.java.atguigu.net/">
 			<arg0>1</arg0>
 		</ns2:getStudentById>
 	</S:Body>
 </S:Envelope>
2. 将上面的xml数据作为请求体，向服务器发送一个http请求（即soap消息， in消息）
3. 服务器端接收到这个请求后，取出请求体的内容
4. 服务器解析请求体，得到数据，并调用对应的web service对象的方法(依据wsdl文档)
5. 将方法执行的返回值，根据wsdl文档生成一个新的xml片断：
 <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
 	<soap:Body>
 		<ns2:getStudentByIdResponse xmlns:ns2="http://server.ws.java.atguigu.net/">
 			<return>
 				<age>12</age>
 				<id>1</id>
 				<name>Jack</name>
 			</return>
 		</ns2:getStudentByIdResponse>
 	</soap:Body>
 </soap:Envelope>
6. 将新xml片断作为响应体，返回给客户端（soap消息， out消息）
7. 客户端接收到响应数据，解析其响应体中的xml片断，封装成student对象
8. 将生成的student对象作为helloWS.getStudentById(1)的返回值，整个请求结束

 3 拦截器
 	为了让程序员能访问或修改soap消息中的内容，cxf设计的拦截器
 	①系统拦截器
 		服务器端拦截器：
 			//通过终端类将helloWS对象发布到address上
 			EndpointImpl endpoint = (EndpointImpl) Endpoint.publish(address, helloWS);
 			//在服务器端添加一个日志的in拦截器
 			endpoint.getInInterceptors().add(new LoggingInInterceptor());
 			//在服务器端添加一个日志的out拦截器
 			endpoint.getOutInterceptors().add(new LoggingOutInterceptor());
 		客户端拦截器：
 			//创建生成SEI实现对象的工厂
 			HelloWSImplService factory = new HelloWSImplService();
 			//得到web service的SEI的实现类对象（动态生成的对象）
 			HelloWS helloWS = factory.getHelloWSImplPort();
 			//得到客户端对象
 			Client client = ClientProxy.getClient(helloWS);
 			//添加客户端的out拦截器
 			client.getOutInterceptors().add(new LoggingOutInterceptor());
 			//添加客户端的in拦截器
 			client.getInInterceptors().add(new LoggingInInterceptor());
 			Student s = helloWS.getStudentById(1);
 			System.out.println("返回 "+s.getName());
 	②自定义拦截器
 		AbstractPhaseInterceptor：抽象过程拦截器，一般自定义的拦截器都会继承于它
 		功能：通过自定义拦截器实现用户名和密码的检查
 			1. 服务器端：
 				设置in拦截器，从soap消息中获取用户名和密码数据，如果不满足条件就不执行web service的方法
 				public class MyIntercept extends AbstractPhaseInterceptor<SoapMessage> {
 					public MyIntercept() {
 						super(Phase.PRE_PROTOCOL);
 					}
 					@Override
 					public void handleMessage(SoapMessage msg) throws Fault {
 						System.out.println("-----handleMessage");
 						
 						Header header = msg.getHeader(new QName("atguigu"));
 						if(header==null) {
 							throw new Fault(new RuntimeException("用户名密码不存在 "));
 						} else {
 							Element data = (Element) header.getObject();
 							if(data==null) {
 								throw new Fault(new RuntimeException("用户名密码不存在22"));
 							} else {
 								String name = data.getElementsByTagName("name").item(0).getTextContent();
 								String password = data.getElementsByTagName("password").item(0).getTextContent();
 								if(!"xfzhang".equals(name) || !"amomo163".equals(password)) {
 									throw new Fault(new RuntimeException("用户名密码不正确"));
 								}
 							}
 						}
 						System.out.println("通过拦截器了！");
 					}
 				}
 			2. 客户端：
 				设置out拦截器，向soap消息中添加用户名和密码数据
 				public class MyIntercept extends AbstractPhaseInterceptor<SoapMessage> {
 					private String name;
 					private String password;
 	
 					public MyIntercept(String name, String pasword) {
 						super(Phase.PRE_PROTOCOL);
 						this.name = name;
 						this.password = pasword;
 	
 					}
 	
 					@Override
 					public void handleMessage(SoapMessage msg) throws Fault {
 						System.out.println("-----handleMessage");
 						Document document = DOMUtils.createDocument();
 						//<atguigu>
 						Element atguiguEle = document.createElement("atguigu");
 						//<name>xfzhang_amomo163</name>
 						Element nameEle = document.createElement("name");
 						nameEle.setTextContent(name);
 						
 						//<password>xfzhang_amomo163</password>
 						Element passwordEle = document.createElement("password");
 						passwordEle.setTextContent(password);
 						
 						atguiguEle.appendChild(nameEle);
 						atguiguEle.appendChild(passwordEle);
 						
 						//添加为请求消息的<soap:Header>
 						msg.getHeaders().add(new Header(new QName("atguigu"), atguiguEle));
 					}
 	
 				}


 			3. 说明：用户名和密码是以xml片断的形式存放在soap消息中的
 				<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
 					<soap:Header>
 						<atguigu>
 							<name>xfzhang</name>
 							<password>amomo163</password>
 						</atguigu>
 					</soap:Header>
 					<soap:Body>
 						<ns2:getStudentById xmlns:ns2="http://server.ws.java.atguigu.net/">
 							<arg0>1</arg0>
 						</ns2:getStudentById>
 					</soap:Body>
 				</soap:Envelope>

 ④用cxf做基于spring的web service开发
 	server端：
 		1. 新建web工程，导入cxf的jar包
 		2. 定义SEI和其实现
 			@WebService
 			public interface OrderService {
 	
 				@WebMethod
 				public Order getOrderByNo(String orderNo);
 			}
 			
 			@WebService
 			public class OrderServiceImpl implements OrderService {
 	
 				public OrderServiceImpl() {
 					System.out.println("OrderServiceImpl()");
 				}
 	
 				@Override
 				public Order getOrderByNo(String orderNo) {
 					System.out.println("getOrderByNo() orderNO=" + orderNo);
 					return new Order(3, orderNo, 10000000);
 				}
 	
 			}
 			
 			public class Order {
 	
 				private int id;
 				private String orderNo;
 				private double price;
 			}
 		3. 发布web service
 			beans.xml
 				<?xml version="1.0" encoding="UTF-8"?>
 				<beans xmlns="http://www.springframework.org/schema/beans"
 					xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 					xmlns:jaxws="http://cxf.apache.org/jaxws"
 					xsi:schemaLocation="http://www.springframework.org/schema/beans  
 									http://www.springframework.org/schema/beans/spring-beans.xsd
 									http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
 					<!-- 
 						配置一些cxf的核心bean
 					 -->			
 					<import resource="classpath:META-INF/cxf/cxf.xml" />
 					<import resource="classpath:META-INF/cxf/cxf-extension-soap.xml" />
 					<import resource="classpath:META-INF/cxf/cxf-servlet.xml" />
 					
 					<!-- 
 						终端：发布webservice(将SEI的实现类对象与web service所提供的网络地址关联)
 					 -->
 					<jaxws:endpoint id="orderService" implementor="net.atguigu.java.ws_spring.ws.OrderServiceImpl"
 						address="/orderService" />
 	
 				</beans>
 			web.xml
 				<?xml version="1.0" encoding="UTF-8"?>
 				<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">
 				  <display-name>day101_ws_spring</display-name>
 				  <welcome-file-list>
 					<welcome-file>index.html</welcome-file>
 					<welcome-file>index.htm</welcome-file>
 					<welcome-file>index.jsp</welcome-file>
 					<welcome-file>default.html</welcome-file>
 					<welcome-file>default.htm</welcome-file>
 					<welcome-file>default.jsp</welcome-file>
 				  </welcome-file-list>
 				  
 				  <!-- 配置上下文初始化参数：指定cxf的spring的beans.xml -->
 				  <context-param>
 					  <param-name>contextConfigLocation</param-name>
 					  <param-value>classpath:beans.xml</param-value>
 				   </context-param>
 				   
 				   <!-- 加载上面参数的配置文件的listener -->
 				   <listener>
 					  <listener-class>
 						 org.springframework.web.context.ContextLoaderListener
 					  </listener-class>
 				   </listener>
 				   
 				   <!-- 匹配所有请求，将请求先交给cxf框架处理 -->
 				   <servlet>
 					  <servlet-name>CXFServlet</servlet-name>
 					  <servlet-class>
 						 org.apache.cxf.transport.servlet.CXFServlet
 					  </servlet-class>
 					  <load-on-startup>1</load-on-startup>
 				   </servlet>
 				   <servlet-mapping>
 					  <servlet-name>CXFServlet</servlet-name>
 					  <url-pattern>/*</url-pattern>
 				   </servlet-mapping>
 				   
 				</web-app>
 	
 	client端
 		1. 新建web工程，导入cxf的jar包
 		2. 根据wsdl文档生成客户端代码（src下）
 		3. client-beans.xml
 				<beans xmlns="http://www.springframework.org/schema/beans"
 				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 				xmlns:jaxws="http://cxf.apache.org/jaxws"
 				xsi:schemaLocation="http://www.springframework.org/schema/beans  
 								http://www.springframework.org/schema/beans/spring-beans.xsd
 								http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
 				<!--
 					配置发送请求的客户端
 				-->
 				<jaxws:client id="orderClient" serviceClass="net.atguigu.java.ws_spring.ws.OrderService"
 					address="http://localhost:8080/day101_ws_spring/orderService" />
 			</beans>
 		4. 编写测试代码：
 			public static void main(String[] args) {
 				//加载client-beans.xml
 				ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
 						"client-beans.xml");
 				//根据bean的id加载所对应的OrderService
 				OrderService orderService = (OrderService) context.getBean("orderClient");
 				//调用方法，发送soap请求
 				Order order = orderService.getOrderByNo("hao123");
 				System.out.println("client " + order.getId() + "_" + order.getOrderNo()
 						+ "_" + order.getPrice());
 			}
 5. 添加拦截器
  客户端:
  	<jaxws:client id="orderClient" serviceClass="com.atguigu.day01_ws_server.ws.OrderWs"
  		address="http://localhost:8989/day02_cxf_spring_server/orderws">
  		<jaxws:outInterceptors>
  			<bean class="com.atguigu.day01_ws_server.ws.intercepter.AddUserIntercepter">
  				<property name="name" value="xfzhang"></property>
  				<property name="pwd" value="123456"></property>
  			</bean>
  		</jaxws:outInterceptors>
  	</jaxws:client>
  服务器端:
  	<jaxws:endpoint id="orderWs"
  		implementor="com.atguigu.day01_ws_server.ws.OrderWsImpl" address="/orderws">
  		<jaxws:inInterceptors>
  			<bean class="com.atguigu.day01_ws_server.intercepter.CheckUserIntercepter"/>
  		</jaxws:inInterceptors>
  	</jaxws:endpoint>
  补充：
 6. 用ajax访问webservice
  a. 使用原生的ajax : ajax.jsp
  b. 使用jquery的ajax : ajax2.jsp

 7. 用HttpURLConnection访问web service : HttpServletConnectionTest
    ​		
Web Service好比一个服务供应商，给其他厂家提供基础服务，其他厂家再将这个服务包装成自己的产品或者服务提供给别人或自己使用。既然两个公司需要合作，不可能靠一句话就可以的，就需要一些标准和规范的东西来实现。那么：

SOAP 就像两个公司之间签的合同，约束双方按一定规矩和标准办事。

WSDL 则像说明书，告诉别人你有什么，能给别人提供什么服务。


​	
​	
​	
​	
​	
​	
​	