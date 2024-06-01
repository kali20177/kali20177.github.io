# 链表


## 普通链表（非侵入式）

由数据域和 link 域组成。

单向：

```c
struct list_element {
    void *data;                    // data 域
    struct list_element * next;    // link 域
};
```

双向：

```c
struct list_element {
    void *data;
    struct list_element * prev;
    struct list_element * next;
};
```

环形：

末尾元素指向开头，Linux 内核标准链表采用双向环形链表，但是不使用非侵入式实现。

**非侵入式链表在一条链表上元素类型必须相同，泛化能力差。**

## 侵入式链表 (intrusive linked lists)

丢弃 data 域：

```c
struct list_head {
    struct list_element * prev;
    struct list_element * next;
};
```

将 list_head 嵌入要操作的数据结构中：

```c
struct ListNodeInt {
    list_head link;
    int data;
}

struct ListNodeDouble {
    list_head link;
    double data;
}
```

因此侵入式链表节点类型不需要一致，使用时需要使用者自己转型。

使用宏 `container_of()` 寻找从链表指针到父结构的变量，在 C 中，结构的偏移在编译时被 ABI 固定。

```c
#define container_of(ptr, type, member) ({          \
     const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
     (type *)( (char *)__mptr - offset_of(type,member) );})
```

通过 `container_of()` 宏返回包含 `list_head` 父类型结构体。

```c
#define list_entry(ptr, type, member) \
        container_of(ptr, type, member)
```

例子：

```c
#include <stdio.h>
#include <stdlib.h>

// 定义 list_head 结构体
struct list_head {
    struct list_head *next, *prev;
};

// 初始化链表头宏
#define INIT_LIST_HEAD(ptr) do { \
    (ptr)->next = (ptr); (ptr)->prev = (ptr); \
} while (0)

// 添加节点的内部函数
static inline void __list_add(struct list_head *new,
                              struct list_head *prev,
                              struct list_head *next) {
    next->prev = new;
    new->next = next;
    new->prev = prev;
    prev->next = new;
}

// 在链表头部添加节点
static inline void list_add(struct list_head *new, struct list_head *head) {
    __list_add(new, head, head->next);
}

// 删除节点的内部函数
static inline void __list_del(struct list_head *prev, struct list_head *next) {
    next->prev = prev;
    prev->next = next;
}

// 删除节点
static inline void list_del(struct list_head *entry) {
    __list_del(entry->prev, entry->next);
    entry->next = entry->prev = NULL;
}

// 获取包含链表节点的结构体指针的宏
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)

#define container_of(ptr, type, member) ({          \
    const typeof( ((type *)0)->member ) *__mptr = (ptr);    \
    (type *)( (char *)__mptr - offsetof(type, member) );})

// 获取链表节点对应的结构体指针
#define list_entry(ptr, type, member) \
    container_of(ptr, type, member)

// 遍历链表的宏
#define list_for_each(pos, head) \
    for (pos = (head)->next; pos != (head); pos = pos->next)

// 定义包含链表节点的结构体
struct my_data {
    int value;
    struct list_head list;
};

// 初始化链表头
struct list_head my_list;

// 添加节点
void add_node(struct list_head *head, int value) {
    struct my_data *new_node = malloc(sizeof(struct my_data));
    new_node->value = value;
    INIT_LIST_HEAD(&new_node->list);
    list_add(&new_node->list, head);
}

// 删除节点
void delete_node(struct my_data *node) {
    list_del(&node->list);
    free(node);
}

// 查找节点
struct my_data* find_node(struct list_head *head, int value) {
    struct list_head *pos;
    list_for_each(pos, head) {
        struct my_data *entry = list_entry(pos, struct my_data, list);
        if (entry->value == value) {
            return entry;
        }
    }
    return NULL;
}

// 打印链表
void print_list(struct list_head *head) {
    struct list_head *pos;
    list_for_each(pos, head) {
        struct my_data *entry = list_entry(pos, struct my_data, list);
        printf("%d ", entry->value);
    }
    printf("\n");
}

int main() {
    INIT_LIST_HEAD(&my_list);

    add_node(&my_list, 10);
    add_node(&my_list, 20);
    add_node(&my_list, 30);

    printf("List after adding nodes: ");
    print_list(&my_list);

    struct my_data *node = find_node(&my_list, 20);
    if (node) {
        printf("Found node with value: %d\n", node->value);
        delete_node(node);
    }

    printf("List after deleting node with value 20: ");
    print_list(&my_list);

    return 0;
}
```

## linked list good taste

内容来自 [linked list good taste](https://github.com/mkirchner/linked-list-good-taste)

// TODO:
