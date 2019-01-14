---
title: selenium中的window handle
date: 2019-01-14 14:04:37
top: 1
tags: 
	- 测试
categories: 
	- 软件测试
---
> webdriver之window handle
> =======================
> 
> 1.  > 实例化一个webdriver后相当于开启一个浏览器进程，  
>     > 一个实例化的driver可以有多个window窗口，在浏览器中显示为多个标签，  
>     > 比如点击一个链接 [网易](163.com)，会打开一个新的窗口
>     
> 2.  > webdriver类中的所有方法有一个前提条件是：都作用于某一window handle
>     
> 3.  > window handle是惰性的，不会自动切换，如果打开了一个新的窗口，想在新窗口上获取某一元素，需要先手动切换window handle，driver.switch\_to.window(xxx\_handle)
>     
> 4.  > 用driver.window_handles可以获取所有窗口句柄
>     
> 5.  > 窗口句柄是浏览器拥有的，元素没有窗口句柄
>     
> 
> window handle示例
> ---------------
> 
>     `import time
>                 from selenium import webdriver
>                 from selenium.webdriver.common.action_chains import ActionChains
> 
>                 def demo():
>                     driver = webdriver.Chrome()
> 
>                     driver.implicitly_wait(10)
> 
>                     driver.get('[http://baidu.com](http://baidu.com)')
> 
>                     print(driver.window_handles) #打开百度后第一次打印窗口句柄
> 
>                     bd_kw = driver.find_element_by_css_selector('#kw')
> 
>                     bd_sb = driver.find_element_by_css_selector('#su')
> 
>                     ac = ActionChains(driver)
> 
>                     ac.send_keys_to_element(bd_kw, 'python').click(bd_sb).perform()
> 
>                     py = driver.find_element_by_xpath('//*[@id="2"]/h3/a')
> 
>                     py.click() #在百度中搜索python后打开一个新的窗口
> 
>                     print(driver.window_handles) #第二次打印窗口句柄
> 
>                     time.sleep(5)
> 
>                     driver.close() #关闭driver的当前句柄，可以用current_handle查看
> 
>                     print(driver.window_handles) #第三次打印窗口句柄
> 
>                     driver.switch_to.window(driver.window_handles[-1]) #切换window handle
> 
>                     print(driver.current_window_handle) #打印current_window_handle，不切换会报异常，因为之前的window已经被我们关闭了
> 
>                     time.sleep(5)
> 
>                     driver.quit()
> 
> 
>                 if __name__ == '__main__':
> 
>                     demo()
> 
>                 ----------------------
>                 >>>['CDwindow-3711170FE14EB6A64A8D9A51249D8EF6'] #只打开了百度首页，所以只有一个
>                 >>>['CDwindow-3711170FE14EB6A64A8D9A51249D8EF6', 'CDwindow-1FDDE8A60F9569D82F5A477DCBF6B8E1'] #打开了百度首页和某一个搜索出来的页面，新的页面在新的窗口中，所以有两个
>                 >>>['CDwindow-1FDDE8A60F9569D82F5A477DCBF6B8E1'] #没切换handle，关闭了第一个window，所以看到，原列表中的第一个元素被删除了，只有新的窗口handle保留下来了
>                 >>>CDwindow-1FDDE8A60F9569D82F5A477DCBF6B8E1 #切换了handle，并打印出current_window_handle` 
> 
> tips：
> -----
> 
> 1.  driver的current handle也是惰性的，如果current window handle被关闭，那么current\_handle这个值就取不到了，会报异常，需要用手动调用driver.switch\_to.window 来显示切换。
> 2.  如果元素定位失败，要检查一下是不是打开了新的窗口，如果是，则需要切换window handle，因为它不会自动切换。