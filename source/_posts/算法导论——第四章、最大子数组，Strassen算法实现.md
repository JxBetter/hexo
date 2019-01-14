---
title: 算法导论——第四章、最大子数组，Strassen算法实现
date: 2019-01-14 13:06:42
top: 1
tags: 
	- algorithm
categories: 
	- algorithm
---
>     
>     #最大子数组
>     def find_max_crossing_subarray(list, low, high):
>         left_sum = -float('inf')
>         right_sum = -float('inf')
>         index = ((high + low) // 2)
>         lsum = 0
>         rsum = 0
>         if len(list) == 1:
>             return (low, low, list[low])
>         if len(list) == 2:
>             return (low, high, list[low] + list[high])
>         for i in range(index, low - 1, -1):
>             lsum += list[i]
>             if lsum > left_sum:
>                 left_sum = lsum
>                 max_left = i
>         for j in range(index + 1, high + 1):
>             rsum += list[j]
>             if rsum > right_sum:
>                 right_sum = rsum
>                 max_right = j
>         return (max_left, max_right, left_sum + right_sum)
>     
>     def find_max_subarray(list, low, high):
>         """
>         :param list: a list
>         :param low: list lowest index, 0
>         :param high: high index, len(list)-1
>         :return: order list
>         """
>         if low == high:
>             return (low, high, list[low])
>         else:
>             mid = (low + high) // 2
>             (left_low, left_high, left_sum) = find_max_subarray(list, low, mid)
>             (right_low, right_high, right_sum) = find_max_subarray(list, mid + 1, high)
>             (cross_low, cross_high, cross_sum) = find_max_crossing_subarray(list, low, high)
>     
>             if left_sum >= right_sum and left_sum >= cross_sum:
>                 return (left_low, left_high, left_sum)
>             elif right_sum >= left_sum and right_sum >= cross_sum:
>                 return (right_low, right_high, right_sum)
>             else:
>                 return (cross_low, cross_high, cross_sum)
>     
> 
>     
>     #Strassen矩阵乘法
>     from numpy import array, dot, hstack, vstack
>     def strassen_matrix(a, b):
>         """
>         a,b must be n*n matrix, n is 2**p, 2,4,8,16...
>         :param a:
>         :param b:
>         :return: a dot b
>         """
>         aa = array(a)
>         bb = array(b)
>         n = aa.shape[0]
>         A11 = []
>         A12 = []
>         A21 = []
>         A22 = []
>         B11 = []
>         B12 = []
>         B21 = []
>         B22 = []
>         if n == 1:
>             return a*b
>         if aa.shape[0] & (aa.shape[0] - 1) != 0:
>             return -1
>         if aa.shape != bb.shape:
>             return -1
>         if aa.shape[0] != aa.shape[1]:
>             return -1
>         for i in range(n // 2):
>             A11.append(a[i][0:n // 2])
>             A12.append(a[i][n // 2:])
>             B11.append(b[i][0:n // 2])
>             B12.append(b[i][n // 2:])
>         for j in range(n // 2, n):
>             A21.append(a[j][0:n // 2])
>             A22.append(a[j][n // 2:])
>             B21.append(b[j][0:n // 2])
>             B22.append(b[j][n // 2:])
>     
>         A11 = array(A11)
>         A12 = array(A12)
>         A21 = array(A21)
>         A22 = array(A22)
>         B11 = array(B11)
>         B12 = array(B12)
>         B21 = array(B21)
>         B22 = array(B22)
>     
>         S1 = B12 - B22
>         S2 = A11 + A12
>         S3 = A21 + A22
>         S4 = B21 - B11
>         S5 = A11 + A22
>         S6 = B11 + B22
>         S7 = A12 - A22
>         S8 = B21 + B22
>         S9 = A11 - A21
>         S10 = B11 + B12
>     
>         P1 = strassen_matrix(A11, S1)
>         P2 = strassen_matrix(S2, B22)
>         P3 = strassen_matrix(S3, B11)
>         P4 = strassen_matrix(A22, S4)
>         P5 = strassen_matrix(S5, S6)
>         P6 = strassen_matrix(S7, S8)
>         P7 = strassen_matrix(S9, S10)
>     
>         C11 = P5 + P4 - P2 + P6
>         C12 = P1 + P2
>         C21 = P3 + P4
>         C22 = P5 + P1 - P3 - P7
>     
>         return (vstack((hstack((C11, C12)), hstack((C21, C22)))))
>