# darc_transfer.php 臺大資料轉換metadata &amp; dublin_core資料轉換程式

<pre>
<?php
		//呼叫PHPExcel檔
		//================================================================================================
		header("Content-type: text/html; charset=utf-8"); //utf-8
		require 'C:\AppServ\www\PHPExcel\Classes\PHPExcel.php'; //加入 PHPExcel.php 檔
		//require 'C:\xampp\htdocs\PHPExcel\Classes\PHPExcel.php'; //加入 PHPExcel.php 檔
	
		//設定記憶體空間和執行時間
		//================================================================================================
		ini_set('memory_limit', '-1'); //加大記憶體空間
		set_time_limit(2000);  
		ini_set("upload_max_filesize","400M"); 
		ini_set("post_max_size","500M"); 
		$ts = microtime(true);
		
		//載入Excel檔案
		//================================================================================================
		$objReader = PHPExcel_IOFactory::createReader('Excel5'); //讀取舊版 Excel 
		$objReader->setReadDataOnly(true); //設定唯讀 
		$objPHPExcel = $objReader->load('dibaosai.xls'); //載入 metadata 檔 
		$objWorksheet = $objPHPExcel->getActiveSheet(); //載入 worksheet 
		$highestRow = $objWorksheet->getHighestRow(); //總列數 
		$highestColumn = $objWorksheet->getHighestColumn(); //總行數 
		$highestColumnIndex = PHPExcel_Cell::columnIndexFromString($highestColumn); //總行數索引
		
		//過濾特殊字元
		//================================================================================================
		$_filter = array("\t", "\0", "\x0B");
		$_replace = array('&', '<', '>', '"', "'");
		$_to = array('&amp;', '&lt;', '&gt;', 'ʹʹ', 'ʹ');
		
		//設定來源檔輸出輸出檔
		//================================================================================================
		function inputUI($t){
			if($t!="" && preg_match('/\D{1}\:\/(.*)\//',$t,$v)){
				fwrite(STDOUT, "Hello, Your copy destination location is $t !\n");
			}
			else{
				fwrite(STDERR, "You not input path or wrong path!\n");
				fwrite(STDOUT, "Enter your location again: (eg.M:/passer_test5/)\n");
				$p = trim(fgets(STDIN));
				inputUI($p);
			}
		}
		
		//設定來源檔輸出檔
		//================================================================================================
		//透過 標準輸出 印出要詢問的內容
		fwrite(STDOUT, "Enter your copy destination location: (eg.C:/AppServ/www/DARC_2012_FINAL/special/dibaosai_2129/)\n");
		// 抓取 標準輸入 的 內容
		$_out = trim(fgets(STDIN));
		// 將剛剛輸入的內容, 送到 標準輸出
		inputUI($_out);
		$_out = mb_convert_encoding($_out,'big5', 'utf8');
		
		//透過 標準輸出 印出要詢問的內容
		fwrite(STDOUT, "Enter your copy source location: (eg.C:/AppServ/www/DARC_2012_SRC/special/dibaosai/)\n");
		// 抓取 標準輸入 的 內容
		$_src = trim(fgets(STDIN));
		// 將剛剛輸入的內容, 送到 標準輸出
		inputUI($_src1);
		$_src = mb_convert_encoding($_src,'big5', 'utf8');
	
		//透過 標準輸出 印出要詢問的內容
		fwrite(STDOUT, "Enter your Full_Metedata.xml <Name> row number: (Please input number eg.1)\n");
		// 抓取 標準輸入 的 內容
		$name = trim(fgets(STDIN));
		
		//透過 標準輸出 印出要詢問的內容
		fwrite(STDOUT, "Enter your Full_Metedata.xml <Display> row number: (Please input number eg.1)\n");
		// 抓取 標準輸入 的 內容
		$display = trim(fgets(STDIN));
		
		//透過 標準輸出 印出要詢問的內容
		fwrite(STDOUT, "Enter your dublin_core.xml element row number: (Please input number eg.2)\n");
		// 抓取 標準輸入 的 內容
		$element = trim(fgets(STDIN));
		
		//透過 標準輸出 印出要詢問的內容
		fwrite(STDOUT, "Enter your dublin_core.xml mainTitle row number: (Please input number eg.3)\n");
		// 抓取 標準輸入 的 內容
		$maintitle = trim(fgets(STDIN));
		
		//字碼轉換函式
		//================================================================================================
		function get_basename($filename)  
		{  	
			return preg_replace('/^.+[\\\\\\/]/', '', $filename);	
		}	
		
		// 讀取標頭列//<=======修改的
		// metadata[0] 為 Full_Metedata.xml 用 <Name>
		// metadata[1] 為 Full_Metedata.xml 用 <Display>
		// metadata[2] 為 dublin_core.xml 第 1 層
		// metadata[3] 為 dublin_core.xml 第 2 層
		//================================================================================================
		$metadata = array();
		for ($row = 0; $row <= 3; $row++)
		{
			for ($col = 0; $col < $highestColumnIndex; $col++)
			{
				$v = str_replace($_filter, '', $objWorksheet->getCellByColumnAndRow($col, $row)->getValue());
				$v = ($v) ? $v : 'none';
				$metadata[$col][] = $v;
			}
		}
		
		//一列一列產生檔案	
		//================================================================================================
		echo '<table>' . "\n";
		for ($row = 4; $row <= $highestRow; ++$row)   
		{	
			//檔案資料夾編號 
			$value = $objWorksheet->getCellByColumnAndRow(0, $row)->getValue();	 
			$value = str_replace($_filter, '', $value);
			$value = trim($value);	
			//echo $value;	
			
			//新建資料夾 
			$path1 = $_out . $value; 
			get_basename($path1); 
			mkdir($path1, 0700); 
						
			//開啟檔案 Full_Metadata.xml	
			$filename1 = $_out . $value . '/Full_Metadata.xml'; 
			get_basename($filename1);
			$fp1 = fopen($filename1, 'w+');	
			fseek($fp1, 0);	
			fwrite($fp1, '<?xml version="1.0" encoding="UTF-8"?>'."\n".'<Item>'."\n"); 
			
			//開啟檔案 dublin_core.xml		
			$filename2 = $_out . $value . '/dublin_core.xml';
			get_basename($filename2);			
			$fp2 = fopen($filename2, 'w+');	
			fseek($fp2, 0);		
			fwrite($fp2, '<?xml version="1.0" encoding="UTF-8"?>'."\n".'<dublin_core>'."\n"); 
			
			//開啟檔案 contents		
			$filename3 = $_out . $value . '/contents';
			get_basename($filename3);
			$fp3 = fopen($filename3, 'w+');	
			fseek($fp3, 0);					
			fwrite($fp3, 'Full_Metadata.xml'."\n");	
			echo '<tr>' . "\n"; 
			
			//============================一行一行輸入產生 contents 檔案===============================================				
			$do = scandir($_src.$value);
			var_dump($do);
			
			for ($i = 2; $i < sizeof($do); $i++)
			{
				if ($do[$i] != 'Thumbs.db')
				{
					echo $do[$i]."\n";
					$c = copy($_src.$value.'/'.$do[$i], $_out.$value.'/'.$do[$i]);
					if($c == true){fwrite($fp3, $do[$i] . "\n");}
				}
			}
			
			//============================一行一行輸入項目=============================================================						
			for ($col = 0; $col < $highestColumnIndex; ++$col) 
			{				 		
				$item = $objWorksheet->getCellByColumnAndRow($col, $row)->getValue(); 
				$item = str_replace($_filter, '', $item);
				$item = str_replace($_replace, $_to, trim($item));
							
				echo '<td>' . $item . '</td>' . "\n";

				//============================一行一行輸入產生 Full_Metadata.xml 檔案===============================================											
				if($item=="")
				{
					continue;
				}
				else
				{	
					fwrite($fp1, '<Field>'."\n");	
					fwrite($fp1, '<Name>'.$metadata[$col][$name].'</Name>'."\n");			
																		
					fwrite($fp1, '<Value>'.$item.'</Value>'."\n");			
																		
					fwrite($fp1, '<Display>'.$metadata[$col][$display].'</Display>'."\n");
					fwrite($fp1, '</Field>'."\n");
				}		
					
				//============================一行一行輸入產生 dublin_core.xml 檔案=================================================				
				if($item=="")
				{
					continue;
				}
				else if($item=="0")
				{
					continue;
				}
				else
				{									
					fwrite($fp2, '<dcvalue element="'.$metadata[$col][$element].'" qualifier="');
				
					$value2 = $metadata[$col][$maintitle];
					
					if($value2 == "")
					{
						fwrite($fp2,'none'.'">');												
					}
					elseif($value2 == "mainTitle" || $value2 == "maintitle")
					{
						fwrite($fp2,'mainTitle'.'">');
					}
					else
					{
						fwrite($fp2,$value2.'">');
					}
					fwrite($fp2, $item.'</dcvalue>'."\n");	
				}
			}	
			
			fwrite($fp1, '</Item>'."\n");
			fclose($fp1);				
			
			fwrite($fp2, '</dublin_core>'."\n");
			fclose($fp2);
			
			fclose($fp3);
			
			echo '</tr>' . "\n";
		}		
	echo '</table>' . "\n";
	
	//顯示所花費時間
	echo 'done, '. (microtime(true) - $ts) .'s';
	
?>
</pre>
