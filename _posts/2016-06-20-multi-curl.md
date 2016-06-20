---
layout: post
title: "curl多线程数据抓取"
description: ""
category: 
tags: [curl]
---

> 做程序开发的时候，经常会遇到产品汪要求你抓取某某网站的数据，我本人是很反感这样的需求的，但是呢，既然是强需求，代码还得撸。

----
    程序抓取有很多种方式
    1.简单粗暴 ：file_get_contents();file_put_contents();
    2.fread(),fwrite(),fopen(),feof(),fgets(),fclose()等一一系列文件操作函数。
    3.curl函数。

----
ps:我在这里主要写了一点curl多线程的代码，仅供参考。

```
$dbOperate = new DbOperate();
//每次接受参数定量抓取
//$limit = $argv[2],$argv[1] ;

$offset = ($argv[2]-1) * $argv[1];
$limit =  $offset.','.$argv[1] ;
$urllist = $dbOperate->getUrlList('cat2=20 ',$limit);   //获取燃油滤清器的链接

if(!empty($urllist)){
    main($urllist,$dbOperate);
}

function main($urllist,$dbOperate){
    if(!empty($urllist)){
        // 初始化 curl_multi
        $mh = curl_multi_init(); 
        $handle = array();
        // 将单个curl 放入队列
        foreach($urllist as $i=>$urldetail){
            $url = $urldetail['sp_url'];
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, $url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1); 
            curl_setopt($ch, CURLOPT_TIMEOUT, 40);
            curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0)');
            curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1); // 302 redirect
            curl_setopt($ch, CURLOPT_MAXREDIRS, 10);
            curl_multi_add_handle($mh, $ch); 
            $handle[$i][0] = $ch;
            $handle[$i][1] = $urldetail['id'];
            $handle[$i][2] = $urldetail['sp_id'];
        }
        do {
            $mrc = curl_multi_exec($mh, $active);
        } while ($mrc == CURLM_CALL_MULTI_PERFORM);
        
        while ($active && $mrc == CURLM_OK) {
        
            if (curl_multi_select($mh) == -1) {
                usleep(1000);
            }
            do {
                $mrc = curl_multi_exec($mh, $active);
            } while ($mrc == CURLM_CALL_MULTI_PERFORM);
                   
        }
        // 实际处理
        foreach($handle as $ch) {
            $httpinfo = curl_getinfo( $ch[0] );
            if($httpinfo['http_code'] == 200){
                $html  = curl_multi_getcontent($ch[0]);

                /*截取网页内容
                $pos = strpos($html,'<div class="g_detail" id="comment">');
                $substr = strstr($html,'<div class="c_foot1">',true);
                $content = substr($substr,$pos);
                */

                //将商品详情页的有效的html内容保存到本地文件。
                $savePath = dirname(__FILE__).'/detail/'.$ch[2].'.html';
                echo $savePath.PHP_EOL;
                $fp = fopen($savePath,'w+');
                $status = fwrite($fp,$html);

                echo $status ? $ch[2].'-- is ok.'.PHP_EOL : " fail to get detail .".PHP_EOL;

            }elseif($httpinfo['http_code'] == 404){
                echo 'the goods is not found.';
            }else{
                echo 'error page.';
            }
        }
        // 移除队列的curl
        foreach($handle as $ch) {
           curl_multi_remove_handle($mh, $ch[0]);
        }
        curl_multi_close($mh);
    }
}

```
{% include JB/setup %}
