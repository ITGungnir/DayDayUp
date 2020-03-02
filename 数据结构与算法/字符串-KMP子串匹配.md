## 字符串-KMP子串匹配

参考：[从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)

`KMP算法`是由三位老前辈：`D.E.Knuth`、`J.H.Morris`、`V.R.Pratt`研究发明的，因此简称为KMP算法。KMP算法的核心是避免不必要的回溯。

KMP算法中的一些术语：
* 文本串S：给定的母字符串；
* 模式匹配串T：给定的子字符串；
* 失配：当T中下标为i的元素与S中对应的元素不相同，且T中下标为1..(i-1)的元素都与S中对应元素相同，则称模式匹配串T在i处失配；
* next数组：当模式匹配串T失配的时候，next数组对应的元素用来指导应该用T中的哪个元素进行下一轮的匹配。

KMP算法的完整代码：
```java
import java.util.Scanner;

public class KMP {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("请输入母串：");
        String mStr = scanner.nextLine();
        System.out.print("请输入子串：");
        String cStr = scanner.nextLine();
        int index = searchStringWithKMP(mStr, cStr);
        if (index < 0) {
            System.out.println("抱歉，母串中不包含子串！");
        } else {
            System.out.println("子串在母串中的位置：" + index);
        }
    }

    private static int searchStringWithKMP(String motherString, String childString) {
        if (motherString.length() < childString.length()) {
            return -1;
        }
        char[] s = motherString.toCharArray();
        char[] t = childString.toCharArray();
        if (s.length == t.length) {
            int i = 0;
            for (; i < s.length; i++) {
                if (s[i] != t[i]) {
                    return -1;
                }
            }
            if (i == s.length) {
                return 0;
            }
        }
        int[] next = generateNextArray(t);
        int si = 0, ti = 0;
        while (si < s.length && ti < t.length) {
            if (ti == -1 || s[si] == t[ti]) {
                si++;
                ti++;
            } else {
                ti = next[ti];
            }
        }
        if (ti == t.length) {
            return si - ti;
        }
        return -1;
    }

    /**
     * 生成K数组（next数组）
     */
    private static int[] generateNextArray(char[] childCharArray) {
        int[] next = new int[childCharArray.length];
        next[0] = -1;
        int i = 0, j = -1;
        while (i < childCharArray.length - 1) {
            if (j == -1 || childCharArray[i] == childCharArray[j]) {
                i++;
                j++;
                if (childCharArray[i] == childCharArray[j]) {
                    next[i] = next[j];
                } else {
                    next[i] = j;
                }
            } else {
                j = next[j];
            }
        }
        return next;
    }
}
```
