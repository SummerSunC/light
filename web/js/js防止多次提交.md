# JS 中阻止多重提交

``` js
var subLock = false;
_self.subPass = function(_data){
	if(subLock) return;	//加锁，防止多重提交
	
	subLock = true;
	$.ajax({
		type:"post",
		url:"/admin/code/pass",
		data:_data,
		dataType:"json"
	}).done(function(res){
		subLock = false;
		if(fxt.isTrue(res.success)){
			fxt.alert("保存成功");
			$("#findForm").submit();
		}else{
			$("#modal_code_num_warn").modal("hide");
			fxt.alert("保存失败");
		}
	})
}
```