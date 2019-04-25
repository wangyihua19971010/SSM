# 基于SSM框架的图片上传
    SSM框架自行配置

###     bug 提示：
    1.数据库图片存储字段长度不够长
    
    2.photo文件夹没有同步到target里：
        在photo下创建文件
        
    3.上传图片是到服务器的：
        controller：
        路径：
        String ss="D:\\自己笔记\\java-1807\\第三阶段项目\\ziruTest\\target\\oaTest-1.0-SNAPSHOT\\personage\\photo\\";
        所以项目存放路径有所改变时就要改变ss(直接复制target下的photo的绝对路径)
        
    4.前端展示图片的src的路径为： "photo/"+图片名
        如：$("#tphoto1").attr("src","photo/"+responseText.photo);    
        
    5.ajax的url错误：
        根据图片判断；
        ajax前端传到后台，看能不能在控制台打印出来，其他事情不要做，返回值随便给个，不能的话只能不断修改路径；
        如：
        @RequestMapping(value = "/personage/uploadFile", produces = "text/html;charset=utf-8")
        @ResponseBody
        public Object importPicFile1() {
        System.out.println("Hello world！！！");
        return "";
        }
        
    6.参数传输错误：
        与5差不多，只不过5好才能进6
        如：
        @RequestMapping(value = "/personage/uploadFile", produces = "text/html;charset=utf-8")
        @ResponseBody
        public Object importPicFile1(
        @RequestParam MultipartFile file1,@RequestParam int id) {
        System.out.println(file1);
        System.out.println(id);
        return "";
        }
            
        
    
    
---

### pom.xml：
<!--https://mvnrepository.com/artifact/commons-io/commons-io -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>

---

###     controller

     /**
     * 图片修改保存
     * @param file1
     * @param id
     * @return
     */
    @RequestMapping(value = "/personage/uploadFile", produces = "text/html;charset=utf-8")
    @ResponseBody
    public Object importPicFile1(
            @RequestParam MultipartFile file1,@RequestParam int id) {
        Map<String, Object> map = new HashMap<String, Object>();
        UserInfoVo userInfoVo = new UserInfoVo();
        userInfoVo.setId(id);
        String falg="";
        File fdir=null;
        String originalFilename = file1.getOriginalFilename();
        String uuid = UUID.randomUUID().toString();
        String ss="D:\\自己笔记\\java-1807\\第三阶段项目\\ziruTest\\target\\oaTest-1.0-SNAPSHOT\\personage\\photo\\";
        String s = uuid.replaceAll("-", "");

        if (file1.isEmpty()) {
            map.put("result", "error");
            map.put("msg", "上传文件不能为空");
        } else {
            String fileBaseName = FilenameUtils.getBaseName(originalFilename);
            Date now = new Date();
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String floderName = fileBaseName + "_" + df.format(now);
            String date = floderName.split(" ")[0];
            originalFilename = s+date+originalFilename;
            try {
                //去掉“-”符号
                //创建要上传的路径
                fdir = new File(ss);
                if (!fdir.exists()) {
                    fdir.mkdirs();
                }
                //文件上传到路径下
                FileUtils.copyInputStreamToFile(file1.getInputStream(), new File(fdir, originalFilename));
                map.put("result", "success");

            } catch (Exception e) {
                map.put("result", "error");
                map.put("msg", e.getMessage());
            }
        }
        userInfoVo.setPhoto(originalFilename);
        boolean b = userService.uploadFile(userInfoVo);
        if (b==true)
            falg="true";

        return falg;
    }

    
    
---
    
###     Data.html

    <script  type="text/javascript" src="js/jquery-1.8.3.min.js"></script>
    <script type="text/javascript" src="js/ajaxfileupload.js"></script>


    <script type="text/javascript">
        // var id = window.location.search.split("=")[1];
            var id = 1;
            //图片与个人信息回显
            $.ajax({
                type: 'POST',
                url: '/lookUser',
                contentType: "application/json; charset=utf-8",
                data:JSON.stringify({
                    "id" : id
                }),
                success: function (responseText) {
                    console.log(responseText);
                    $("#tphoto1").attr("src","photo/"+responseText.photo);
                    $("#tphoto2").attr("src","photo/"+responseText.photo);
                    $("#tphoto3").attr("src","photo/"+responseText.photo);
                    $("#nickname").val(responseText.name);
                    $("#tusername").val(responseText.username);
                    $("#temail").val(responseText.email);
                    //3.1清空表格
                    // $("#datad1  tr:not(:first)").empty("");
                    $(responseText).each(function (index, item) {
                        // alert(item.username);
                        $(  "<tr>"+
                                "<td>"+ item.username +
                                "</td>"+
                            "</tr>").insertAfter($("#datad1 tr:eq(0)"));
                    })
                    //3.1清空表格
                    // $("#datad2  tr:not(:first)").empty("");
                    $(responseText).each(function (index, item) {
                        // alert(item.username);
                        $(  "<tr>"+
                                "<td>"+ item.username +
                                "</td>"+
                            "</tr>").insertAfter($("#datad2 tr:eq(0)"));
                    })
    
    
                },
    
                error: function (message) {
                    console.log(message);
                },
                dataType: 'json'
            });
    
        //上传图片
        function imageUpload(){
            var file1 = document.getElementById("file1");
            var ssFile = document.getElementById("ssFile");
            ssFile.value = file1.value.substring(12);
            var img = document.getElementById("tphoto1");
            // alert(img.src);
            var url = img.src;
            console.log(ssFile.value);//取出文件名，并赋值回显到文本框，用于向后台传文件名
            $.ajaxFileUpload({
                url : 'uploadFile?id='+id, //用于文件上传的服务器端请求地址
                fileElementId : 'file1', //文件上传空间的id属性  <input type="file" id="file" name="file" />
                type : 'post',
                dataType : 'text', //返回值类型 一般设置为json
                success : function(imageUrl) //服务器成功响应处理函数
                {
                    console.log(imageUrl);
                    if(imageUrl==true){
                        location.href="Space.html";
                    }
                },
                error : function(data, status, e)//服务器响应失败处理函数
                {
                    alert("文件上传失败");
                }
            });
    //修改保存
            $.ajax({
                type: 'POST',
                url: 'upload',
                contentType: "application/json; charset=utf-8",
                data:JSON.stringify({
                    "id":id,
                    "username":$("#tusername").val(),
                    "name":$("#nickname").val(),
                    "email":$("#temail").val()
                }),
                success:function (response) {
                    if(response==true){
                        window.location.href = "Space.html";
                    }else{
                        alert("修改失败");
                    }
    
                },
                dataType: 'json',
                error: function (message) {
                    console.log(message);
                }
            })
        }
    
    </script>
    
    <script type="text/javascript">
        //单纯图片显示
        function uploadImg(fileDom) {
            // 获取图片数据对象
            var file = fileDom.files[0];
            //对图片内容进行base64编码
            var reader = new FileReader();
            reader.readAsDataURL(file);
    
            //确保文件成功获取，base64数据量比较大
            reader.onload = function (event) {
                var e = event || window.event;
                var img = document.getElementById("tphoto1");
                img.src = e.target.result;
                //或者 img.src = this.result; 因为e.target == this
                // alert(img.src);
            }
    
        }
    </script>




    <body>
         <img  src=""   width="20" class="img" id="tphoto3">
            <table id="datad1">
                <tbody>
                    <tr>
                        <td>
                        </td>
                    </tr>
                </tbody>
            </table>     


    <div class="img">
        <img src="" width="100" height="100" id="tphoto2" >
    </div>
        <table id="datad2" align="center" style="font-size:24px;">
            <tbody>
                <tr>
                    <td>
                    </td>
                </tr>
            </tbody>
        </table>
    </body>



---     
    
###     springmvc-servlet.xml
            
    <!-- 文件上传的解析器 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 上传图片的最大尺寸 10M-->
        <property name="maxUploadSize" value="10485760"></property>
        <!-- 默认编码 -->
        <property name="defaultEncoding" value="utf-8"></property>
    </bean>
            
            
            
