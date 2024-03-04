## Grade

| Labs          | 8% (24 points)      |
| :------------ | :------------------ |
| Homework      | 12% (36 points)     |
| **Project 1** | **10% (30 points)** |
| **Project 2** | **10% (30 points)** |
| **Project 3** | **10% (30 points)** |
| **Project 4** | **10% (30 points)** |
| Midterm       | 16% (48 points)     |
| Final         | 24% (72 points)     |



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

`(-1)^S * (1+significand) * 2^(exponent-127)`

