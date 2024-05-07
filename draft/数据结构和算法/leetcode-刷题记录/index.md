# Leetcode 刷题记录


## 2023.09.17

1. [1768] 交替合并字符串

    ```c
    char * mergeAlternately(char * word1, char * word2) {
        int len1 = strlen(word1);
        int len2 = strlen(word2);

        char* res = malloc((strlen(word1) + strlen(word2) + 1) * sizeof(char));
        memset(res, '\0', (strlen(word1) + strlen(word2) + 1));

        int i = 0, j = 0, k = 0;
        while (j < len1 && k < len2) {
            if (i % 2 == 0) 
                res[i] = word1[j], j ++;
            else 
                res[i] = word2[k], k ++;
            i++;
        }

        if (word1[j] != '\0') {
            while (j < len1) res[i] = word1[j], j++, i++;
        }

        if (word2[k] != '\0') {
            while (k < len2) res[i] = word2[k], k++, i++;
        }
        
        return res;
    }
    ```

2. [206] 反转链表

    ```c
    struct ListNode* reverseList(struct ListNode* head){
        struct ListNode* pre = NULL;  // 前驱节点，记录链表信息，最后作为结果返回
        struct ListNode* cur = head;  // 新建 cur 节点，避免操作 head

        while (cur != NULL) {
            struct ListNode* tmp = cur -> next;  // 保存断开节点后的信息
            cur -> next = pre;
            pre = cur;
            cur = tmp;
        }
        return pre;
    }
    ```

    {{< bilibili id=BV1KZ4y157Up >}}

3. [26] 删除有序数组

    ```c
    int removeDuplicates(int* nums, int numsSize){
        int index = 0; // 记录数组实际不重复数组长度

        // j < numsSize
        for (int i = 1; i < numsSize; i++) {
            if (nums[i] != nums[index]) {
                nums[++index] = nums[i];
            }
        }
        return index + 1;
    }
    ```

4. [234] 回文链表

    - C 语言遍历链表到数组
    - C 语言的双指针

    ```c
    bool isPalindrome(struct ListNode* head){
        int vals[1000001], index = 0;
        while (head != NULL) {
            vals[index++] = head -> val;
            head = head -> next;
        }

        for (int i = 0, j = index - 1; i < j; ++i, --j) {
            if (vals[i] != vals[j]) return false;
        }
        return true;
    }
    ```

5. [1832] 判断句子是否为全字母句

    ```c
    bool checkIfPangram(char * sentence){
        int map[26] = { 0 };
        int len = strlen(sentence);
        for (int i = 0; i < len; i ++) {
            map[sentence[i] - 97]++;
        }
        for (int j = 0; j < 26; j++) {
            if (map[j] == 0)
                return false;    
        }
        return true;
    }
    ```

6. [2828] 判别首字母缩略词

    ```c
    bool isAcronym(char ** words, int wordsSize, char * s) {
        char res[101] = "";
        for (int i = 0; i < wordsSize; i++) {
            res[i] = words[i][0];
        }
        if (strcmp(res, s) == 0) 
            return true;
        else 
            return false;
    }
    ```

    1. C 语言中字符串数组

        ```c
        char str[3] = {"AA", "B", "CCC"};
        ```

        [C 语言字符串数组的两种表示方法](https://blog.csdn.net/LSKCGH/article/details/78047143)

        作为参数传入：

        ```c
        bool isAcronym(char ** words, int wordsSize, char * s) 
        ```

    2. 比较字符串

        ```c
        if (strcmp(res, s) == 0) 
            return true;
        ```

