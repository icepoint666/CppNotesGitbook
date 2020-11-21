# 代码整洁之道（Java示例）

* 合适的命名
* 短小的方法
* 减少嵌套if/else语句
* 抽离try/catch
* 封装多个函数参数

### 合适的命名

先把业务弄清楚，组织会议定下常用业务领域的单词，禁止组员各自发明。比如代码里使用`canteen`表示饭堂，那就不要再发明`DinnerHall`，既啰嗦又误导同僚。

### 短小的方法

过长的方法，让阅读者不知道从何看起，看了前面忘记后面。将复杂的方法，拆分成逻辑相对简单的短方法。

**看看反例：**

```java
//  获取个人信息
Private UserDTO getUserDTO(Integer userId)
{
    //获取基本信息 
    … 此处写了10行
​
    //获取最近的一次订单信息
    …  此处写了30行
​
   // 获取钱包余额、可用优惠券张数等
    ...   此处写了30行
​
   return userDTO;
}
```

**看看正例：**

```java
//  获取个人信息
Private UserDTO getUserDTO(Integer userId)
{
    //获取基本信息 
    UserDTO userDTO= getUserBasicInfo(userId);
​
    //获取最近的一次订单信息
    userDTO.setUserLastOrder(getUserLastOrder(userId));
​
    // 获取钱包、可用优惠券张数等
    userDTO.setUserAccount(getUserAccount(userId));  
    return userDTO;
}
​
Private  UserDTO getUserBasicInfo(userId);
Private  UserLastOrder getUserLastOrder(userId);
Private  UserAccount getUserAccount(userId);
```

### 减少if/else嵌套

为什么要减少嵌套，难道嵌套看上去不时尚吗？我曾经看到某位同事的一段代码嵌套达到9层，他自己再去维护的时候都看晕了。代码过度嵌套的结果是只有原作者才能读懂，接盘侠一脸茫然。

**看看反例：**

```java
// 修改用户密码，这个例子只有3层嵌套，很温柔了
public boolean modifyPassword(Integer userId, String oldPassword, String newPassword) {
      if (userId != null && StringUtils.isNotBlank(newPassword) && SpringUtils.isNotBlank(oldPassword)) {
    User user = getUserById(userId);
    if(user != null) {
         if(user.getPassword().equals(oldPassword) {
              return updatePassword(userId, newPassword)
         }
    }
      }
}
```

**看看正例：**

```java
// 修改用户密码 
Public Boolean modifyPassword(Integer userId, String oldPassword, String newPassword) {
     if (userId == null || StringUtils.isBlank(newPassword) || StringUtils.isBlank(oldPassword)) {
            return false;
     }
     User user = getUserById(userId);
     if(user == null) {
           return false;
      }
     if(!user.getPassword().equals(oldPassword) {
           return false;    
     }
     return updatePassword(userId, newPassword);
}
```

正例采用卫语句减少了嵌套，但是并非所有场景都适合这样改写。如果不适合，可以将关联性高的逻辑抽取成一个独立的方法减少嵌套。

### 抽离try/catch

大家有没有见过一个超长的方法，从头到尾被一个try/catch照顾着？博主经历过的项目中，这种不负责的写法比比皆是。并非每行代码都会抛出错误，只要将会抛出错误的业务放在一个独立的方法即可。

**看看反例：**

```java
//  获取个人信息
Private UserDTO getUserDTO(Integer userId)
{
   try { 
       //获取基本信息 
       ... 此处写了10行
       //获取最近的一次订单信息.
       ...此处写了20行
       // 获取钱包、可用优惠券张数等
       ...此处写了20行
    }catch (Exception e) {
        logger.error(e);
        return null;
    }
}
   return userDTO;
}
```

**看看正例：**

```java
//  获取个人信息
Private UserDTO getUserDTO(Integer userId)
{
    //获取基本信息 
    UserDTO userDTO= getUserBasicInfo(userId);
​
    //获取最近的一次订单信息
    userDTO.setUserLastOrder(getUserLastOrder(userId));
​
    // 获取钱包、可用优惠券张数等
    userDTO.setUserAccount(getUserAccount(userId));  
    return userDTO;
}
Private  UserDTO getUserBasicInfo(userId);
Private  UserLastOrder getUserLastOrder(userId);
Private  UserAccount getUserAccount(userId)｛
      try{ // TODO } catch( Exception e) { //TODO}
｝
```

### 封装多个参数

如果方法参数将超过3个，建议放在类中包装起来，否则再增加参数时，由于语义的强耦合会导致调用方语法错误。在后台管理中的分页查询接口，常常会有很多查询参数，而且有可能增加，封装起来是最好的。

**看看反例：**

```java
// 分页查询订单 6个参数
Public Page<Order> queryOrderByPage(Integer current,Integer size,String productName,Integer userId,Date startTime,Date endTime,Bigdecimal minAmount ,Bigdecimal maxAmount) {
​
}
```

**看看正例：**

```java
@Getter
@Setter
Public class OrderQueryDTO extends PageDTO {
 private String productName;
 private Integer userId;
 private Date startTime;
 private Date endTime;
 private Bigdecimal minAmount ;
 private Bigdecimal maxAmount;
}
// 分页查询订单 6个参数
Public Page<Order> queryOrderByPage(OrderQueryDTO orderQueryDTO) {

}
```

