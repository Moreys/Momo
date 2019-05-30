# TinyXML2 学习笔记
## 1、概述
- 开源的C++ XML解析库，合适配置简单数据、配置文件，对象实例化等不适数据量很大的操作

## 2、用法
- xml文件本质就是小型的数据库，
- 换个角度来说就是，对数据库有什么操作，那么对xml文件就应能实现什么操作。
- 对数据库的操作包括以下几种：
  - 新建数据库和对数据库增删查改。
  - 那么对应xml文件就是新建xml文件
  - 增加xml文件的节点，删除xml文件的指定节点，
  - 查询xml文件指定节点的值，
  - 修改xml文件中节点的值。

### 2.1组成
- 声明
  - Version   版本 
  - Standalone  
  - Encoding  文本解析
- 根节点
  - 创建节点文件

- 其他节点

### 2.2 创建
```c++
//用户类
class User
{
public:
    User(){
        gender=0;
    };

    User(const string& userName, const string& password, int gender, const string& mobile, const string& email){
        this->userName=userName;
        this->password=password;
        this->gender=gender;
        this->mobile=mobile;
        this->email=email;
    };

    string userName;
    string password;
    int gender;
    string mobile;
    string email;
};

//function:insert XML node
//param:xmlPath:xml文件路径; user:用户对象
//return:0:成功; 非0:失败
int insertXMLNode(const char* xmlPath,const User& user)
{
    XMLDocument doc;
    int res=doc.LoadFile(xmlPath);
    if(res!=0)
    {
        cout<<"load xml file failed"<<endl;
        return res;
    }
    XMLElement* root=doc.RootElement();

    XMLElement* userNode = doc.NewElement("User");
    userNode->SetAttribute("Name",user.userName.c_str());
    userNode->SetAttribute("Password ",user.password.c_str());
    root->InsertEndChild(userNode);

    XMLElement* gender = doc.NewElement("Gender");
    XMLText* genderText=doc.NewText(itoa(user.gender));
    gender->InsertEndChild(genderText);
    userNode->InsertEndChild(gender);

    XMLElement* mobile = doc.NewElement("Mobile");
    mobile->InsertEndChild(doc.NewText(user.mobile.c_str()));
    userNode->InsertEndChild(mobile);

    XMLElement* email = doc.NewElement("Email");
    email->InsertEndChild(doc.NewText(user.email.c_str()));
    userNode->InsertEndChild(email);

    return doc.SaveFile(xmlPath);
}
```

### 2.3、获取节点
- 一个用户节点存储一个用户的信息。因此，
- 对用户信息的增删查改，即无论查询节点、删除节点、修改节点和增加节点，都需要获取需要操作的节点。
- 实现一个根据用户名获取节点指针的函数：
```c++
//function:根据用户名获取用户节点
//param:root:xml文件根节点；userName：用户名
//return：用户节点
XMLElement* queryUserNodeByName(XMLElement* root,const string& userName)
{

    XMLElement* userNode=root->FirstChildElement("User");
    while(userNode!=NULL)
    {
        if(userNode->Attribute("Name")==userName)
            break;
        userNode=userNode->NextSiblingElement();//下一个兄弟节点
    }
    return userNode;
}
```
- 在上面函数的基础上，获取用户的信息函数
```c++
User* queryUserByName(const char* xmlPath,const string& userName)
{
    XMLDocument doc;
    if(doc.LoadFile(xmlPath)!=0)
    {
        cout<<"load xml file failed"<<endl;
        return NULL;
    }
    XMLElement* root=doc.RootElement();
    XMLElement* userNode=queryUserNodeByName(root,userName);

    if(userNode!=NULL)  //searched successfully
    {
        User* user=new User();
        user->userName=userName;
        user->password=userNode->Attribute("Password");
        XMLElement* genderNode=userNode->FirstChildElement("Gender");
        user->gender=atoi(genderNode->GetText());
        XMLElement* mobileNode=userNode->FirstChildElement("Mobile");
        user->mobile=mobileNode->GetText();     
        XMLElement* emailNode=userNode->FirstChildElement("Email");
        user->email=emailNode->GetText();           
        return user;
    }
    return NULL;
}
```






### 2.4、修改XML文件的指定节点
```c++
//function:修改指定节点内容
//param:xmlPath:xml文件路径；user：用户对象
//return：bool
bool updateUser(const char* xmlPath,User* user)
{
    XMLDocument doc;
    if(doc.LoadFile(xmlPath)!=0)
    {
        cout<<"load xml file failed"<<endl;
        return false;
    }
    XMLElement* root=doc.RootElement();
    XMLElement* userNode=queryUserNodeByName(root,user->userName);

    if(userNode!=NULL)
    {
        if(user->password!=userNode->Attribute("Password"))
        {
            userNode->SetAttribute("Password",user->password.c_str());  //修改属性
        }
        XMLElement* genderNode=userNode->FirstChildElement("Gender");
        if(user->gender!=atoi(genderNode->GetText()))  
        {
            genderNode->SetText(itoa(user->gender).c_str());   //修改节点内容
        }
        XMLElement* mobileNode=userNode->FirstChildElement("Mobile");
        if(user->mobile!=mobileNode->GetText())
        {
            mobileNode->SetText(user->mobile.c_str());
        }
        XMLElement* emailNode=userNode->FirstChildElement("Email");
        if(user->email!=emailNode->GetText())
        {
            emailNode->SetText(user->email.c_str());
        }
        if(doc.SaveFile(xmlPath)==0)
            return true;
    }
    return false;
}
```

- 验证代码
```c++
int main(int argc,char* argv[])
{
    //修改用户信息
    User user("lvlv","00001111",0,"13995648666","1586666@qq.com");
    if(updateUser("./user.xml",&user))
        cout<<"update successfully"<<endl;
    else
        cout<<"update failed"<<endl;
    return 0;
}
```

### 2.5、删除XML文件的指定节点信息
```c++
//function:删除指定节点内容
//param:xmlPath:xml文件路径；userName：用户名称
//return：bool
bool deleteUserByName(const char* xmlPath,const string& userName)
{
    XMLDocument doc;
    if(doc.LoadFile(xmlPath)!=0)
    {
        cout<<"load xml file failed"<<endl;
        return false;
    }
    XMLElement* root=doc.RootElement();
    //doc.DeleteNode(root);//删除xml所有节点
    XMLElement* userNode=queryUserNodeByName(root,userName);
    if(userNode!=NULL)
    {
        userNode->DeleteAttribute("Password");//删除属性
        XMLElement* emailNode=userNode->FirstChildElement("Email");
        userNode->DeleteChild(emailNode); //删除指定节点
        //userNode->DeleteChildren();//删除节点的所有孩子节点
        if(doc.SaveFile(xmlPath)==0)
            return true;
    }
    return false;
}
```
- 验证代码
```c++
int main(int argc,char* argv[])
{
    //删除用户某些信息
    if(deleteUserByName("./user.xml","lvlv"))
        cout<<"delete successfully"<<endl;
    else
        cout<<"delete failed"<<endl;
    return 0;
}
```

## 3、其他常见用法
### 3.1、获取XML文件的声明
```c++
//function:获取xml文件申明
//param:xmlPath:xml文件路径；strDecl：xml申明
//return：bool
bool getXMLDeclaration(const char* xmlPath,string& strDecl)
{
    XMLDocument doc;
    if(doc.LoadFile(xmlPath)!=0)
    {
        cout<<"load xml file failed"<<endl;
        return false;
    }
    XMLNode* decl=doc.FirstChild();  
    if (NULL!=decl)  
    {  
        XMLDeclaration* declaration =decl->ToDeclaration();  
        if (NULL!=declaration)  
        {  
              strDecl = declaration->Value();
              return true;
        } 
    }
    return false;
}
```
- 验证代码
```c++
int main(int argc,char* argv[])
{
    //获取xml文件申明
    string strDecl;
    if(getXMLDeclaration("./user.xml",strDecl))
    {
        cout<<"declaration:"<<strDecl<<endl;
    }
    return 0;
}
```

### 3.2、打印XML文件至标准输出
```c++
//function:将xml文件内容输出到标准输出
//param:xmlPath:xml文件路径
void print(const char* xmlPath)
{
    XMLDocument doc;
    if(doc.LoadFile("./user.xml")!=0)
    {
        cout<<"load xml file failed"<<endl;
        return;
    }
    doc.Print();
}
```

### 3.3、XML文件内容输出至内存
```c++
XMLPrinter printer;
doc.Print( &printer );
// printer.CStr() has a const char* to the XML
```