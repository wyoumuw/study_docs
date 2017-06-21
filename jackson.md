# Jackson from Jackson official tutorial
* Streaming [doc](https://github.com/FasterXML/jackson-core/wiki) [tutorial](https://github.com/FasterXML/jackson-core)
* Annotations [doc](https://github.com/FasterXML/jackson-annotations/wiki) [tutorial](https://github.com/FasterXML/jackson-annotations)
* Databind [doc](https://github.com/FasterXML/jackson-databind/wiki) [tutorial](https://github.com/FasterXML/jackson-databind)

## Databind

### 使用
#### 基本使用
	//这里省略Person类的定义
	ObjectMapper mapper = new ObjectMapper();//创建一次重复使用
	Person person=mapper.readValue([File|byte[]|String..],Person.class);//把json转成参数2的类型，这个函数有很多的重载
	String personJson=mapper.writeValueAsString(person);//Object转String
	byte[] personBytes=mapper.writeValueAsBytes(person);//Object转byte[]
	mapper.writeValue([OutputStream|File..],person);//把Object写成json流出到参数1
	
#### 泛型集合和树模型（tree model）
	Map<String,Integer> map=mapper.readValue(json,Map.class);
	List<String> list=mapper.readValue(json,List.class);
	//转Json
	String listString=mapper.writeValueAsString(list);
	mapper.writeValue([OutputStream|File..],list);
	//当集合里是复杂的内容则需要做泛型引用
	Map<String, Person> results = mapper.readValue(jsonSource,new TypeReference<Map<String, Person>>() { } );
	//TreeModel的使用，可以自己构造Json
	ObjectNode root = mapper.readTree("stuff.json");
	String name = root.get("name").asText();
	int age = root.get("age").asInt();
	// 能简单的修改: 给root添加对象类型的属性'other',设置其属性'type'
	root.with("other").put("type", "student");
	String json = mapper.writeValueAsString(root);
	// 上面的json将呈现如下
	// {
	//   "name" : "Bob", "age" : 13,
	//   "other" : {
	//      "type" : "student"
	//   }
	// }
### 5分钟介绍: Streaming parser, generator
其实就是流模型，他处理json成对象或者处理对象成json。他是Databind和TreeModel的底层处理。

	JsonFactory f = mapper.getFactory(); // 也可以直接构建

	//创建一个到处目的文件
	File jsonFile = new JsonFile("test.json");
	//生成Generator
	JsonGenerator g = f.createGenerator(jsonFile);
	// 写一个 JSON: { "message" : "Hello world!" }
	g.writeStartObject();
	g.writeStringField("message", "Hello world!");
	g.writeEndObject();
	g.close();

	// 第二步：将文件读回来
	JsonParser p = f.createParser(jsonFile);

	JsonToken t = p.nextToken(); // 应该为 JsonToken.START_OBJECT
	t = p.nextToken(); // JsonToken.FIELD_NAME
	if ((t != JsonToken.FIELD_NAME) || !"message".equals(p.getCurrentName())) {
	   // 处理错误
	}
	t = p.nextToken();
	if (t != JsonToken.VALUE_STRING) {
	   // 同上
	}
	String msg = p.getText();
	System.out.printf("My message to you is: %s!\n", msg);
	p.close();
	//以上只是示例，具体参考streaming的文档
### 10分钟介绍: 配置（configuration）
这里有两种配置Features和Annotations（）

### Features
#### Mapper Features
这些配置是ObjectMapper的配置，修改会一直影响ObjectMapper，这些配置配置了POJO内部详情，和把生成了的对象(serializers, deserializers, related)进行缓存。如果你想要不同的配置请分离不同的ObjectMapper进行使用。
##### 使用方法
	mapper.configure(MapperFeature,boolean);

设置可以分为以下几类：
##### 注解处理
* MapperFeature.USE_ANNOTATIONS---- 默认是true，是否使用annotations。

##### 自省，属性检测

* MapperFeature.AUTO_DETECT_CREATORS----默认是true，在没有显示的使用@JsonCreator情况下会自动检测构建方法(构造函数，返回this类型的工厂方法等)。
 会被检测的构建方法有：
 * + public修饰
 * + 单参数并且类型是String|int|long|boolean
 * + 对于工厂方法,函数名必须为valueOf其他的方法不会被检测。
* MapperFeature.AUTO_DETECT_FIELDS----默认是true,如果关闭就不会从public field取值，此功能优先级低于注解。
* MapperFeature.AUTO_DETECT_GETTERS----默认是true,如果关闭就不会通过getter来取值，此功能优先级低于注解。
* MapperFeature.AUTO_DETECT_IS_GETTERS----默认是true,如果关闭boolean的命名isXXX就不会被检测到。
* MapperFeature.AUTO_DETECT_SETTERS----默认是true
* MapperFeature.REQUIRE_SETTERS_FOR_GETTERS----默认是false
* MapperFeature.USE_GETTERS_AS_SETTERS----默认是true
* MapperFeature.INFER_PROPERTY_MUTATORS----默认是true (since 2.2)
 +  
* ALLOW_FINAL_FIELDS_AS_MUTATORS----默认是true (since 2.2)

##### 反射，访问
* CAN_OVERRIDE_ACCESS_MODIFIERS----默认是true
* OVERRIDE_PUBLIC_ACCESS_MODIFIERS----默认是true (since 2.7)

##### 名字处理
* SORT_PROPERTIES_ALPHABETICALLY----默认是false
* USE_WRAPPER_NAME_AS_PROPERTY_NAME----默认是false
* ACCEPT_CASE_INSENSITIVE_PROPERTIES----默认是false
* USE_STD_BEAN_NAMING----默认是false

##### 其他
* USE_STATIC_TYPING----默认是false
* DEFAULT_VIEW_INCLUSION----默认是true
* IGNORE_DUPLICATE_MODULE_REGISTRATIONS----默认是true

#### SerializationFeature(序列化Feature)
Jackson定义了一组与序列化相关的功能（将Java对象编写为JSON），并且可以在每个调用的基础上（通过使用ObjectWriter）进行更改。 
例如：

    final ObjectWriter w = objectMapper.writer();
    // enable one feature, disable another
    byte[] json = w
      .with(SerializationFeature.INDENT_OUTPUT)
      .without(FAIL_ON_EMPTY_BEANS.WRAP_EXCEPTIONS)
      .writeValueAsBytes(value);
	
你也可以改变ObjectMapper的默认配置,通过enable(feature)开启，disable(feature) 和configure(feature,state)来修改。
设置有以下分类：
##### 通用输出Feature
* WRAP_ROOT_VALUE----默认是false,如果为true会加一层并基于@JsonRootNam|@XmlRootElement|无包名类名作为键值，如Person关闭情况会生成{"name":"youmu"}但是开启后就会生成{"Person":{"name":"youmu"}}
* INDENT_OUTPUT----默认是false,输出成多行(pretty-mode)
* 

	
	
	