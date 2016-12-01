在深圳现场出现过这个问题，是由 servlet-api 版本不一致导致的。开发时使用的版本是3.0(由tomcat提供)，而部署时使用的2.5(由weblogic提供)。

上传文档的一个 Servlet 里面，用到了 **ServletRequest** 的 **getServletContext** 方法，如下所示：
```
protected void doPost(HttpServletRequest request, HttpServletResponse response){
    DiskFileItemFactory factory = new DiskFileItemFactory();
    ServletFileUpload fileUpload = new ServletFileUpload(factory);
    fileUpload.setSizeMax(1024 * 1025 * 1024);
    //文件上传地址
    String filePath = this.getServletContext().getRealPath("/")+"fileUpload";
    //... doSomeThing
}
```
getServletContext 这个方法是在3.0中新增的，所以在2.5的情况下，肯定会报NoSuchMethodError这个错误。因此，我们需要改成下面的写法：
```
protected void doPost(HttpServletRequest request, HttpServletResponse response){
    DiskFileItemFactory factory = new DiskFileItemFactory();
    ServletFileUpload fileUpload = new ServletFileUpload(factory);
    fileUpload.setSizeMax(1024 * 1025 * 1024);
    //文件上传地址
    String filePath = this.getSession().getServletContext().getRealPath("/")+"fileUpload";
    //... doSomeThing
}
```
当时的环境是没法调试的，日志也没法看，所以定位问题不像平时那么简单。**这里再次提醒我们开发与部署环境一致性很重要**。

**参考文档**
- [JSR 154: JavaTM Servlet 2.4 Specification](http://download.oracle.com/otn-pub/jcp/7876-servlet-2.4-prd-spec-oth-JSpec/servlet-2_4-prd-spec.pdf?AuthParam=1480574874_443a62f4cde4da7190f3243ee4ece8cc)
- [JSR 315: JavaTM Servlet 3.0 Specification](http://download.oracle.com/otn-pub/jcp/servlet-3.0-public-oth-JSpec/servlet-3.0-pr.pdf?AuthParam=1480574731_6a62fa5f8abc4ca8eedcd20b97823eb5)