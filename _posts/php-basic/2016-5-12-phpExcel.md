---
layout: post
title: "使用phpexcel导入导出数据"
description: ""
category: php-basic
tags: []
---

> 很多后台开发场景中都用到了导入导出功能，所以在这里记录一下，项目背景：CI框架+phpExcel。

### 导入

```
    /**
     * 导入数据
     * @param array $file 文件上传数组
     * @return array $arr 上传的数据
     */
    public function importData($file) {
        ini_set('display_errors', true);
        error_reporting(E_ALL);
        //上传文件
        $time=date("YmdHis");
        $extend = strrchr($file['name'],'.');
        $name=$time.$extend;
        //文件临时目录
        $filePath = NX_ROOT.'application'.DIRECTORY_SEPARATOR.'cache'.DIRECTORY_SEPARATOR;
        $uploadfile = $filePath.$name;
        //移动文件
        $result = move_uploaded_file($file['tmp_name'], $uploadfile);
        //上传成功导入数据
        $arr = array();
        if ($result) {
            $this->load->library('PHPExcel');   //加载扩展类                         
            //$objReader = PHPExcel_IOFactory::createReaderForFile($uploadfile);
            $excelType = PHPExcel_IOFactory::identify($uploadfile); 
            $objReader = PHPExcel_IOFactory::createReader($excelType);
            $objPHPExcel = $objReader->load($uploadfile)->getActiveSheet();
            $highestRow = $objPHPExcel->getHighestRow();  //取得总行数
            $highestColumn = $objPHPExcel->getHighestColumn(); //取得总列数            
            for ($j = 2; $j <= $highestRow ; $j++) {
                for ($k = 'A'; $k <= $highestColumn ; $k++) {                  
                    $arr[$j][] = $objPHPExcel->getCell("$k$j")->getValue();
                }
            }
            @unlink($uploadfile); //删除excel文件
            return $arr; 
        }
    }

```
----

### 导出
  
  导出比较简单，直接指定header头，就没有使用phpExcel扩展插件。

```
    /**
     * 导出数据
     */
    public function exportData()
    {
        //指定header头信息
        header('Content-Type:applicationnd.ms-excel');
        header('Content-Disposition:attachment; filename=channel.xls');
        //查询要导出的数据
        $this->load->model('channel/channel');
        $channel_arr = $this->channel->list_data(); 
        //拼接字符串
        $str = "ID\t车型库\t综述\t车型页\t计算器\t配置\t图片\t降价\t统计\t渠道ID\t渠道名\t使用渠道\t备注\t链接\n";
        if ($channel_arr) {
            foreach ($channel_arr as $k=>$v) 
            {
                if ($v) {
                    $qname = str_replace(chr(10), '', $v['name']);
                    $comment = str_replace(chr(10), '', $v['comment']);
                    $note = str_replace(chr(10),'', $v['note']);
                    $href = TOUCH_URL.'channel/'.$k.'/#zoneclick='.$v['zoneclick'];
                    $str.="{$k}\t{$v['channel']}\t{$v['seriseParent']}\t{$v['carModel']}\t{$v['calculator']}\t {$v['config']}\t{$v['photo']}\t{$v['reduceP']}\t{$v['zoneclick']}\t{$v['qudao_id']}\t{$qname}\t{$comment}\t{$note}\t{$href}\n";
                }
            }
        }
        //如果需要转码就转码，反之直接输出字符串
        echo  mb_convert_encoding( $str ,'gbk','utf-8');
    }

```


