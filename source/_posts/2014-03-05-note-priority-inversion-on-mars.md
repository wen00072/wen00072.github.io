---
layout: post
title: '筆記：Priority Inversion on Mars'
date: 2014-03-05 03:35
comments: true
categories: 筆記
---
* [Priority Inversion on Mars投影片](http://www.slideshare.net/jserv/priority-inversion-30367388)

* 筆記
	* [Readers–writers problem](http://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)
	*	Synchronization: Basic Rules
		*	Process/thread的lock絕對不可互相等待
		* 當lock的順序無法確定時，使用 try_lock or try_unlock 以及對應的恢復方法
		* mutex是critical section專用，除了critical section外process/thread要等待其他的process做完才做事這樣的行為請用semaphore
	* [Live lock](http://en.wikipedia.org/wiki/Deadlock#Livelock)
		*	Wiki範例：兩個有禮貌的人在長廊中互相讓路，讓來讓入大家都卡在長廊裡面。
	* Priority inversion現象之一是在waiting queue中高優先權的thread/process等待低優先權的process/thread裡面的mutex release的現象
	* ASI:  Atmospheric Structure Investigation
	*	MET: Meteorology 
	* Bounded priority inversion
		* 高優先權的process/thread等待進入critical section，該critical section目前由低優先權的process/thread佔用中。因此只要低優先權的process/thread離開該critical section後高優先權的process/thread便可繼續執行
	* Unbounded priority inversion
		* 高優先權的process/thread等待進入critical section，該critical section目前由低優先權的process/thread佔用中
		* 不幸的是，當低優先權process/thread還在critical section執行的時候，被切換到中優先權的process/thread由於高優先權的process/thread被block，而低優先權的process/thread一定會被中優先權的process/thread搶走執行權。最壞的狀況就是之後就只剩中優先權的process/thread被執行
	*	解法？
		* Priority inheritance
			* 當高優先權的process/thread要進入critical section發現該section以被低優先權的process/thread佔用時，系統暫時將該低優先權的process/thread調整到高優先權直到該低優先權的process/thread離開critical section
				* 看來可以解Unbounded priority inversion，bounded priority inversion應該還是本質無法解掉？
	*	除錯觀點
		* 火星探測車錯誤發生的地方是在select的系統實作上有用到mutex，而低優先權的priority使用該API
		* 直接從火星打開偵錯訊息傳回地球太過不切實際。後來確認在地球可以複製錯誤後全力追蹤context switch, 中斷和sync的部份   
