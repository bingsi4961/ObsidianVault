---
date: 2025-06-09 09:59
aliases: []
tags:
  - jQuery
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

```htmlmixed
<!DOCTYPE html>
<html>
<head>
    <title>DataTables 範例</title>
    <!-- jQuery -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    
    <!-- DataTables CSS -->
    <link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.13.7/css/jquery.dataTables.min.css">
    
    <!-- DataTables JS -->
    <script type="text/javascript" src="https://cdn.datatables.net/1.13.7/js/jquery.dataTables.min.js"></script>
	<script type="text/javascript">
		$(document).ready(function() {
			$('#example').DataTable({
				searching: true,
				ordering: true,
				pagingType: "full_numbers", // 設定分頁樣式
				pageLength: 3, 				// 每頁顯示筆數（為了展示分頁效果）
				language: {
					"lengthMenu": "顯示 _MENU_ 筆資料",
					"zeroRecords": "沒有找到符合的資料",
					"info": "顯示第 _START_ 至 _END_ 筆，共 _TOTAL_ 筆資料",
					"infoEmpty": "顯示第 0 至 0 筆，共 0 筆資料",
					"infoFiltered": "(從 _MAX_ 筆資料中過濾)",
					"search": "搜尋:",
					"paginate": {
						"first": "第一頁",
						"last": "最後一頁",
						"next": "下一頁",
						"previous": "上一頁"
					}
				}
			});
		});
	</script>
</head>
<body>
    <table id="example" class="display" style="width:100%">
		<thead>
			<tr>
				<th>姓名</th>
				<th>職位</th>
				<th>部門</th>
				<th>薪水</th>
				<th>入職日期</th>
			</tr>
		</thead>
		<tbody>
			<tr>
				<td>張小明</td>
				<td>軟體工程師</td>
				<td>IT部門</td>
				<td>$65,000</td>
				<td>2020/01/15</td>
			</tr>
			<tr>
				<td>李小華</td>
				<td>專案經理</td>
				<td>IT部門</td>
				<td>$85,000</td>
				<td>2019/05/20</td>
			</tr>
			<tr>
				<td>王大同</td>
				<td>UI/UX設計師</td>
				<td>設計部門</td>
				<td>$55,000</td>
				<td>2021/03/10</td>
			</tr>
			<tr>
				<td>陳小美</td>
				<td>資料分析師</td>
				<td>資料科學部</td>
				<td>$70,000</td>
				<td>2020/08/25</td>
			</tr>
			<tr>
				<td>林志明</td>
				<td>系統管理員</td>
				<td>IT部門</td>
				<td>$60,000</td>
				<td>2019/12/01</td>
			</tr>
		</tbody>
	</table>
</body>
</html>
```