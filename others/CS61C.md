为什么C中使用struct需要用`struct Student student`https://poe.com/s/SlZ9u08r8xPWZ1YBu34F

如果使用typedef,就不再需要使用struct

```c
typedef struct S{
    int id;char* name;
} Student;//换成另外一种形式,就是typedef struct S Student;
...
    Student student;
    struct S student2;
...
```

```c
struct S{
	int id;char* name;
};typedef struct S Student;
...
    struct S stu;
    Student student;
...
```

