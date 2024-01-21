## lambda函数

```c++
[捕获列表](参数){函数体}
sort(dictionary.begin(),dictionary.end(),[&](auto &a,auto &b){
   return (a.size()>b.size())||((a.size()==b.size())&&a<b); 
});
```

# 常见数据类型库

## string

```c++
string s = "Hello";
s.length()或者s.size();//获取字符串长度
s[0] = 'h';//修改单个字符
s = "hello"+" "+"world";//字符串拼接(效率比较低)
```

## vector

```c++
vector<int> ans(len,0);//初始化vector
ans[0] = 10;
ans.push_back(11);
int size = ans.size();
sort(ans.begin(),ans.end(),cmp);

to_string(num);//数字转化为字符串
```

## stack

```c++
#include<stack>
```
- 查看栈顶,弹出栈顶,插入栈顶

```c++
stack<int> s;
s.top();//查看栈顶元素
s.pop();//弹出栈顶元素
s.push(19);//插入元素
```

- 判断非空,遍历,栈大小

```c++
if(s.empty());//判断是否为空
while(!s.empty()){...;s.pop();}
s.size();//栈大小
```

## queue

```c++
#include<queue>
```

- 查看,添加,弹出,遍历,大小 

```c++
queue<node> q;
node top = q.front();
q.push(node(3,1));
q.pop();
while(!q.empty);
q.size();
```

## unordered_set

- 引入:

```c++
#include<unordered_set>
```

- 插入,查找和删除

```c++
unordered_set<int> hashset;
hashset.insert(x);
if(hashset.count(x)>0)......;//注意hashset.count(x)的结果只有可能是0或者1
hashset.erase(0);
```

- 遍历

```c++
for(auto it=hashset.begin();it!=hashset.end();it++){*it = 10;}
for(const auto& element:hashset){...}
```

## unordered_map

- 引入:

```c++
#include<unordered_map>
```

- 插入,修改,查找和删除

```c++
unordered_map<string,int> hashmap;
hashmap["apple"] = 1;
hashmap["pen"] = 2;
hashmap["apple"] += 2;
//查看"grapes"是否存在(0或者1)
if(hashmap.count("grapes")){
	hashmap["grapes"]++;
}//注意就算hashmap["grapes"]不存在,使用之后即便没有赋值也会存在了
hashmap.erase("pen");
```

- 遍历(每个元素是pair的形式)

```c++
for(auto &node:hashmap){
    node.first...;//key
    node.second...;//value
}
for(pair<const string,int>& node:hashmap){...}
for(auto it=hashmap.begin();it!=hashmap.end();it++){
    cout<<(*it).first<<" "<<(*it).second<<endl;
}
```

