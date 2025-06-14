# 剑指Offer 刷题记录

| 题目                                                      | 做题消耗时间 | 刷题时间      | 是否完成              |
| --------------------------------------------------------- | ------------ | ------------- | --------------------- |
| [LCR 001. 两数相除](https://leetcode.cn/problems/xoh6Oh/) | 1.11.42      | 2025年6月12日 | 就是案例一个一个Debug |
|                                                           |              |               |                       |
|                                                           |              |               |                       |

## 已自主完成



## 需要二刷

### [LCR 001. 两数相除](https://leetcode.cn/problems/xoh6Oh/)

这冗余的if，凭着之前做题的记忆，好歹一个一个案例过了，快速幂，快速乘，其实不小心瞟了一眼之前做的答案

INT_MIN需要考虑，转换a，b为负数

```
class Solution {
public:
    int divide(int a, int b) {
         int flag = 1;
        
        if (a == INT_MIN && b==-1){
            return INT_MAX;
        }
        if (a == INT_MIN && b==1){
            return INT_MIN;
        }
        if (a==INT_MIN && b==INT_MIN){
            return 1;
        }
        if (b == INT_MIN){
            return 0;
        }
         if ( a<0&&b>0 ){
            flag = -1;
            b = flag*b;
         }
        if (a>0 && b<0){
            flag = -1;
            a = flag*a;
        }
        if (a>0 && b>0){
            a = -1*a;
            b = -1*b;
        }
        if (a>b || a==0){
            return 0;
        }
       
        int ans = 0;
        int sum = b,max_index=0;
        vector<int>divide_ele;
        
        while(a-sum<=sum){
            divide_ele.push_back(sum);
            sum += sum;
            max_index = max_index+1;
            // cout<<max_index<<" "<<sum<<endl;
        }
        cout<<sum<<endl;
        divide_ele.push_back(sum);
        while(a<=b){
            if (a<=divide_ele[max_index]){
                a-=divide_ele[max_index];
                // cout<<a<<" "<<divide_ele[max_index]<<" "<<max_index<<endl;
                ans+=1<<max_index;
                // cout<<ans<<endl;
                
            }
            max_index = max_index-1;
        }
        return ans*flag;
    }
};
```

