# 最大公约数和最小公倍数

## 问题描述
已知两个正整数a和b，求这两个数的最大公约数和最小公倍数

## 输入
两个正整数a和b

## 输出
GCD(a,b)和LCM(a,b)

## 分析

**GCD(a,b)** = GCD(a,b - a) = GCD(a, b - 2a) = ... = **GCD(a, b % a)**


**LCM(a,b) = a * b / GCD(a,b)**



## 代码实现

	#include <iostream>
	using namespace std;
	
	//a,b是输入的两个数，并且需要a > b
	int gcd(int a,int b) 
	{
		int r = a % b;
		if(r == 0)
			return b;
		else 
			return gcd(a, b % a);
	}
	int main()
	{
		int a,b;
		cin >> a >> b;
		if( a < b)
		{
			int temp = a;
			a = b;
			b = temp;
		}
		int max = gcd(a,b);
		int min = a*b / max;
		cout << "The greatest common divisor is " << max
		 << "and the least common multiple is " << min <<endl;
		return 0;
	}
	

## 求n个数的最大公约数


**性质：**
 
 **GCD(a1,a2,a3) = GCD(GCD(a1,a2), a3);**
 
 **GCD(a1,a2,a3,a4) = GCD(GCD(a1,a2,a3),a4);**
 
 
###代码实现
 
 	#include <iostream>
 	using namespace std;
 	int gcd(int a,int b)
 	{
 		if(a < b)
 		{
 			int temp = a;
 			a = b;
 			b = temp;
 		}
 		int r = a % b;
 		if ( r == 0 )
 			return b;
 		else 
 			return gcd(a,b % a);
 	}
 	int main()
 	{
 		int n;
 		cin >> n;
 		cout << "input n numbers:" << endl;
 		int r;
 		int a,b;
 		cin >> r;
 		for(int i = 1;i <= n-1;i++)
 		{
 			cin >> b;
 			r = gcd(r,b);
 		}
 		cout << "The gcd of n numbers is " << r << endl;
 		return 0;
 	}
 	
 
 
 
