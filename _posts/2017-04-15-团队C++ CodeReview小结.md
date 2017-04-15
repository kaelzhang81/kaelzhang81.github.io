---
layout:     post
title:      "团队C++代码走查小结"
subtitle:   "the summary of C++ code review"
date:       2017-04-15 23:00:00
author:     "Kaelzhang"
header-img: "img/code-bg.jpg"
catalog:    true
tags:
    - code review
    - c++
---

![code review](/img/in-post/team cpp code review/code review.png)

# 团队C++代码走查小结
本文内容源于团队的每日代码走查活动，对照Clean Code基本原则和项目C++编码规范将项目代码中实际出现的一些问题进行提炼归纳，以便经验传播、避免问题反复出现。
PS：由于信息安全已对示例代码进行去业务处理。
### 命名
> 名副其实
> 避免误导
> 做有意义的区分

### 驼峰命名规范



#### 命名大小写规则
<b>
示例：

```
std::string getglobalInstKey(std::string key)
{
	std::stringstream instKey;
	instKey<<getServlet()<<"."<<key;
	return instKey.str();
}
```
其中函数名getglobalInstKey不满足小驼峰命名规则，应该改为：getGlobalInstKey。

#### 函数命名应为动词
可参考编码规范：规则 4.1.3 类名应该是名词或名词短语;接口可以是形容词; 方法名应该是动词或 动词短语
<b>
示例：

```
LmInfo lmInfoInterface(WORD32 gnbId)
{
	LmInfo lm = lmInfoRepository::getInstance().getLmInfo(Id);

	return lm;

}
```
示例中的函数则是一个名词，同样的例子还有**：GlobalLmId LmInfo::globalLm() const;**。

#### 使用统一的名字
可参考编码规范：丰富你的单词库，在面对具体问题时你具有更多的 Colorfull Words
但同一个概念在同一个工程中应该尽量保持一致性
<b>
示例（在预处理的概念上存在多种表示）:

```
preDeal方法
与
preHandle方法
建议：统一使用handle来表达处理这一概念
```

#### 命名应该尽量简洁，没有冗余
可参考编码规范：名字在明确意图的前提下越短越好
<b>
示例：

```
DEFINE_ROLE(BaseProcesser)
{
    BaseProcesser();

    Status process();

private:
    Status doFilter();
}
```
上述示例中的doFilter方法中的do明显是冗余的。

#### 缩写不统一
缩略词的使用上没有完全统一，存在自定义的情况。
<b>
示例：

```
typedef struct  ChanInfo
{
    BYTE        ucChanNo;   
}PACKED TChanInfo
```
```
struct LogicChannMap
{
    LogicChannMap();
} logicChannMap;
```
上述示例中，chan和chann都是channel的缩略词，此处建议使用chan。

#### 常量命名规范
对枚举和const命名比较随意，不满足规则 4.1.1 遵守统一的命名规范的要求。
<b>
示例：

```
enum DataType {
  AMBR = 0,
  TRANS_ID = 1,
  NBID = 2,
  NG = 3,
  RR_INFO = 4,
  GNB = 5,
  RELATIONIDS = 6,
  NfIDS = 7,
  PFID = 8,
};
```
其中NBID、RELATIONIDS、NfIDS、PFID等都不满足常量编码规范。

### 注释
> 最好的注释是没有注释
> 对代码表达欠缺信息的补充
> 不要试图美化代码

#### 便于追踪的注释
由于流程并行开发，现实代码中存在很多打桩的情况，这里最好在所有的注释中加上todo以便后续跟踪、处理。

```
void XXInfo::clear()
{
    ROLE(XXField)->clear();  //todo to verfiy it ;
}
```

### 格式
> 关系密切、概念相关的相互靠近

#### 对齐问题
可参考编码规范：规则 1.1.5 团队统一配置 IDE 的 TAB 为相同数目的空格，坚决抵制使用 TAB 对齐代码
在不同的编辑器下，tab的长度是不一样的，导致格式混乱。
通过下面设置可以使得空格替换tab：
![eclipse tab1](/img/in-post/team cpp code review/eclipse%20tab1.jpg)
![eclipse tab2](/img/in-post/team cpp code review/eclipse%20tab2.jpg)


#### 空格问题
空格该有的地方没有，不该有的地方却有，比较随意。
<b>
示例：

```
WORD16 handle_init_response(Message& event, PbeventInfo& info, WORD16 msgId)
{
    info.setSize(sizeof(InitResponse));
    info.setMsg((void *)&event.init_response());
    INFO_LOG("XXX_inst received: %s[%d].\n"_,_"InitResponse", msgId);
    return _msgId ;_
}
```

### 函数
> 函数要短小、再短小
> 只做一件事
> 每个函数一个抽象层级

#### 多层嵌套的函数
可参考编码规范：规则 7.1.1 避免过长，嵌套过深的函数实现
<b>
示例：

```
OVERRIDE(Status save(const XXXInfoResponse &msg))
{
    xyDatas.clear();
    XInfo info;
    XData data;
    for(auto & xy_info:msg.xy_info_list())
    {
        for(auto & x_info:xy_info.x_info_list())
        {
            info.clear();
            memset(&data, 0, sizeof(data));
            data.val1 = x_info.val1();
            for(int i = 0; i < x_info.val2().size(); ++i)
            {
                data.val2[i].val = x_info.val2(i).val();
            }
            info.insert(make_pair(x_info.key(),data));
        }
        xyDatas.insert(make_pair(xy_info.key(),info));
    }
    return SUCCESS;
}
       
```

#### 平铺直叙的长函数
可参考编码规范：规则 7.1.3 函数中的每一个语句都在一个相同的抽象层次上

```
Status SetVal()
{
    Sample1 sample1;
    sample1->set_val1(1);
    sample1->set_val2(2);
    sample1->set_val3(3);
    sample1->set_val4(4);

    Sample2 sample2;
    sample2->set_val1(1);
    sample2->set_val2(2);
    sample2->set_val3(3);
    sample2->set_val4(4);

    Sample3 sample3;
    sample3->set_val1(1);
    sample3->set_val2(2);
    sample3->set_val3(3);
    sample3->set_val4(4);

    Sample4 sample4;
    sample4->set_val1(1);
    sample4->set_val2(2);
    sample4->set_val3(3);
    sample4->set_val4(4);

    return SUCCESS;
}
```
该函数总共涵盖四个赋值过程，每个过程的细节完全暴露在函数中，因此同时包含了两个不同的层次的代码。建议使用**Extract Method**进行函数提取。改为：

```
Status SetVal()
{
    SetSample1();
    SetSample2();
    SetSample3();
    SetSample4();

    return SUCCESS;
}
```

#### 返回值要合理设计
示例1：
冗余的Status状态返回

```
Status XXXBuilder::build(ReqMsg& req)
{
    auto xxx = ROLE(XXX).globalXXX();

    req.set_id(xxx.getId());
    req.set_result(xxx.getResult());

    return DCM_SUCCESS;
}
```
整个过程中不会出现异常的情况，因此Status返回略显多余。

示例2：
无效的Status状态返回

```
Status XXXRole::req()
{
    AutoMsg<ReqMsg> req;

    ROLE(Sender).send(req.getRef());

    WAIT_ON(EV_RSP, handleRsp);

    return CONTINUE;
}
```
send调用了底层平台send接口，会出现失败的情况，但函数无论send成功失败都返回成功。

### 类
#### 单一职责
示例：

```
DEF_SIMPLE_ASYNC_ROLE(XXXReqRole)
{
    Status config();
    Status reject(const Status cause);
    Status release(const Status cause);

private:
    ROLE_EVENT_HANDLER_DECL(handleReq, XXXReq);

private:
    USE_ROLE(XXXReleaseMsgBuilder);
    USE_ROLE(XXXRejectMsgBuilder);
    USE_ROLE(XXXReleaseMsgSender);
    USE_ROLE(XXXRejectMsgSender);
}
```
上述代码的类名为XXXReqRole，且定义为异步的role, 从接口上看reject和release显然并不属于该类的职责。从实现上看rejcet依赖XXXRejectMsgBuilder和XXXRejectMsgSender, 而release又只依赖XXXReleaseMsgBuilder和XXXReleaseMsgSender,其他依赖都是XXXReqRole的，所以正确的做法应该是分成3个role，每个role只做一件事。如果有必要行成一个聚合根，至少名字上应该更抽象化，例如XXXActor或XXXObject等。

#### 类暴露细节过多
在函数内连续使用另一个类的多个方法的时候，需要考虑类提供的外部接口是否合理。
示例：

```
ROLE_EVENT_HANDLER_DEF(XXXRole, handleRsp, XXXResponse)
{
    USI_ASSERT_TRUE(event.result()==0);
    
    ROLE(RrmRadioResource).updateMain(true);
    ROLE(RrmRadioResource).updateOther(true);
    ROLE(RrmRadioResource).doUpdateMain();
    ROLE(RrmRadioResource).doUpdateOther(;

    return SUCCESS;
}
```

#### 火车失事代码
可参考编码规范：规则 7.1.11 避免实现火车失事的代码

```
req->config()->xxx()->setVal(ROLE(XXXRole).getValue().xxx());
```
连串的调用违反了Demeter法则，所有知识都暴露给了调用方。

### 物理设计
#### 冗余的头文件

```
// #include "haed1andhaed2.h"
#include "haed1.h"

Haed1 haed;
```
所包含的头文件已经被包含在其他头文件中，无需重复包含。

#### 范围过大的头文件

```
// #include "all.h"
#include "part.h"

Part part;
```
所包含的头文件范围大于实际使用的范围，需要进一步缩小依赖范围。

### 重复
#### 逻辑上的重复
项目中存在大量类似但又不完全一样的代码，可以通过OO的多态或者函数式编程的方法来消除。
<bar>
示例：

```
void FakeSystem::wait(string session, string key, const EventId& eventId,const AbstractAsserter& asserter)
{
    reschedure();
    try{

        INFO_LOG( "FakeSystem::wait  event[%d-%s] to msg queue!!\n", eventId, EventName::to_s(eventId));
        auto msg = RECV_MSG_QUEUE.pop(session, key, eventId);

        instKey = msg.instKey();
        session = msg.session();

        asserter.assertMsg(eventId, msg, *this);
    }catch(const Exception e)
    {
        RECV_MSG_QUEUE.dump();
        throw e;
    }
}

void FakeSystem::wait(const EventId& eventId, const AbstractAsserter& asserter)
{
    reschedure();
    try{

        INFO_LOG( "FakeSystem::wait  event[%d-%s] to msg queue!!\n", eventId, EventName::to_s(eventId));
        auto msg = RECV_MSG_QUEUE.pop(eventId, *this);

        instKey = msg.instKey();
        session = msg.session();

        asserter.assertMsg(eventId, msg, *this);
    }catch(const Exception e)
    {
        RECV_MSG_QUEUE.dump();
        throw e;
    }
}
```

可以尝试通过lambda表达式来消除重复的逻辑：

```
void FakeSystem::wait(string session, string key, const EventId& eventId,const AbstractAsserter& asserter)
{
    auto pop = [=](){
        return RECV_MSG_QUEUE.pop(session, key, eventId);
    };

    waitMsg(eventId, asserter, pop);
}

void FakeSystem::wait(const EventId& eventId, const AbstractAsserter& asserter)
{
    auto pop = [=](){
        return RECV_MSG_QUEUE.pop(eventId, *this);
    };

    waitMsg(eventId, asserter, pop);
}

void FakeSystem::waitMsg(const EventId& eventId, const AbstractAsserter& asserter, const std::function<Message()>& pop)
{
    reschedure();
    try{

        INFO_LOG( "FakeSystem::wait  event[%d-%s] to msg queue!!\n", eventId, EventName::to_s(eventId));
        auto msg = pop();

        instKey = msg.instKey();
        session = msg.session();

        asserter.assertMsg(eventId, msg, *this);
    }catch(const Exception e)
    {
        RECV_MSG_QUEUE.dump();
        throw e;
    }
}

```

### 效率提升
#### 不必要的对象赋值

```
CellData lmCellInfoInterface(WORD32 Id)
{
	XXXData xxxData = XXXRepository::getInstance().getXXXInfo(Id);

	return xxxData;
}
```
直接return即可。

#### 右值引用
使用右值引用实现移动语义减少拷贝赋值。
<b>
示例：

```
    XXXInfo xxxInfo(&info);
    xxxInfo.update(val);

    auto var = map.find(info.id);
    DCM_ASSERT_TRUE(var == map.end());

    map[info.id] = std::move(xxxInfo)
```

#### 无序容器
传统的map和set是有序的容器，在插入元素时会对容器进行自动排序，在非排序的场景下使用无序容器unordered_map和unordered_set会更加高效。
<b>
示例：

```
DEF_INTERFACE(If, 0x170323)
{
    Status save();
    Status update(val);
    Info delete(val);

    Status accept(Visitor&) const;

private:
	std::unordered_map<ID, Info> map;
}
```
传统的map的实现使用的是红黑树，而无序map则是使用散列表实现。

### 技巧
#### auto类型推导的使用
通过auto关键字的使用，隐藏不必要的冗长信息。

```
using ns::IdQueryRequest;
using ns::Id;

Status IdQueryRole::build(ns::Message& msg)
{
    IdQueryRequest* req = msg.id_query_request();
    ASSERT_VALID_PTR(req);

    Id* id = req->getId();
    ASSERT_VALID_PTR(id);

    id->set_id(ROLE(InfoField)->id());
    id->set_val(ROLE(IdField)->value());

    return SUCCESS;
}
```

```
~~using ns::IdQueryRequest;
using ns::Id;~~

Status IdQueryRole::build(ns::Message& msg)
{
    auto req = msg.id_query_request();
    ASSERT_VALID_PTR(req);

    auto id = req->getId();
    ASSERT_VALID_PTR(id);

    id->set_id(ROLE(InfoField)->id());
    id->set_val(ROLE(IdField)->value());

    return SUCCESS;
}
```

#### 基于范围的for循环
C++11基于范围的for以统一、简洁的方式来遍历容器和数组。

```
    const Message pop(const EventId& eventId, const FakeSystem& sys)
    {
        typedef std::list<Message>::iterator Iter;
        for (Iter i=msgs.begin(); i != msgs.end(); ++i)
        {
            if ((i->messageId() == eventId) && (sys.expect(*i)))
            {
                Message msg = *i;
                msgs.erase(i);
                return msg;
            }
        }
        onFail(string(), string(), eventId);
    }
```
改为：

```
const Message pop(const EventId& eventId, const FakeSystem& sys)
    {
        for (auto& msg: msgs)
        {
            if (msg.messageId() == eventId) && (sys.expect(msg))
            {
                Message out = *msg;
                msgs.erase(msg);
                return out;
            }
        }
        onFail(string(), string(), eventId);
    }
```
或者结合lambda：

```
    const Message pop(const EventId& eventId, const FakeSystem& sys)
    {
        auto msg = find_if(msgs.begin(), msgs.end(), [&](Message& msg){
            return (msg.messageId() == eventId) && (sys.expect(msg));
        });

        if(out != msgs.end())
        {
            Message out = *msg;
            msgs.erase(msg);
            return out;
        }

        onFail(string(), string(), eventId);
    }
```







