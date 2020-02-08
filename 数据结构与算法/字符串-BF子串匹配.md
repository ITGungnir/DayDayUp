## 字符串-BF子串匹配

BF（Brute Force）子串匹配算法，即暴力比较的方法，其思路是：先找到长字符串中与短字符串中第一个字符相等的字符的位置，再依次向后比较后面的字符是否相等。

例如，有两个字符串：
```text
ILoveJsJackJava.java
Java
```

比较过程如下：
* 遍历第一个字符串，寻找字符`J`，发现下标5的位置是字符`J`；
* 比较第一个字符串中下标6的字符与第二个字符串中下标1的字符，发现不相等，放弃后续比较；
* 继续遍历第一个字符串，寻找字符`J`，发现下标7的位置是字符`J`；
* 比较第一个字符串中下标8的字符与第二个字符串中下标1的字符，发现相等，继续比较；
* 比较第一个字符串中下标9的字符与第二个字符串中下标2的字符，发现不相等，放弃后续比较；
* 继续遍历第一个字符串，寻找字符`J`，发现下标11的位置是字符`J`；
* 比较第一个字符串中下标12的字符与第二个字符串中下标1的字符，发现相等，继续比较；
* 比较第一个字符串中下标13的字符与第二个字符串中下标2的字符，发现相等，继续比较；
* 比较第一个字符串中下标14的字符与第二个字符串中下标3的字符，发现相等，比较完毕，找到子串位置为11。

代码如下：
```java
public class BF {

    public static void main(String args[]) {
        System.out.print("请输入母串：");
        Scanner scanner = new Scanner(System.in);
        String s1 = scanner.nextLine();
        char[] str1 = s1.toCharArray();
        System.out.print("请输入子串：");
        String s2 = scanner.nextLine();
        char[] str2 = s2.toCharArray();

        int index = getSubStringIndex(str1, str2);
        if (index < 0) {
            System.out.println("抱歉没有找到子串");
        } else {
            System.out.println("找到子串，在第" + index + "个位置");
        }
    }

    private static int getSubStringIndex(char[] str1, char[] str2) {
        if (str2.length > str1.length) {
            return -1;
        }
        int index = 0;
        boolean found = false;
        for (; index < str1.length - str2.length + 1; index++) {
            if (str1[index] != str2[0]) {
                continue;
            }
            int j = 1;
            for (; j < str2.length; j++) {
                if (str1[index + j] != str2[j]) {
                    break;
                }
            }
            if (j == str2.length) {
                found = true;
                break;
            }
        }
        return found ? index : -1;
    }
}
```
