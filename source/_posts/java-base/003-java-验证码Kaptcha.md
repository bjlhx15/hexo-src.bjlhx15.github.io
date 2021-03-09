---
title: 003-java-验证码Kaptcha
categories:
  - java-base
abbrlink: a53d4816
date: 2021-01-08 22:53:58
---

摘要：保Kaptcha 是一个可高度配置的实用验证码生成工具，可自由配置的选项如：
- 验证码的字体
- 验证码字体的大小
- 验证码字体的字体颜色
- 验证码内容的范围(数字，字母，中文汉字！)
- 验证码图片的大小，边框，边框粗细，边框颜色
- 验证码的干扰线
- 验证码的样式(鱼眼样式、3D、普通模糊、...)
<!-- more -->

# Kaptcha 详细配置表
```
    kaptcha.border	图片边框，合法值：yes , no	yes
    kaptcha.border.color	边框颜色，合法值： r,g,b (and optional alpha) 或者 white,black,blue.	black
    kaptcha.image.width	图片宽	200
    kaptcha.image.height	图片高	50
    kaptcha.producer.impl	图片实现类	com.google.code.kaptcha.impl.DefaultKaptcha
    kaptcha.textproducer.impl	文本实现类	com.google.code.kaptcha.text.impl.DefaultTextCreator
    kaptcha.textproducer.char.string	文本集合，验证码值从此集合中获取	abcde2345678gfynmnpwx
    kaptcha.textproducer.char.length	验证码长度	5
    kaptcha.textproducer.font.names	字体	Arial, Courier
    kaptcha.textproducer.font.size	字体大小	40px.
    kaptcha.textproducer.font.color	字体颜色，合法值： r,g,b 或者 white,black,blue.	black
    kaptcha.textproducer.char.space	文字间隔	2
    kaptcha.noise.impl	干扰实现类	com.google.code.kaptcha.impl.DefaultNoise
    kaptcha.noise.color	干扰 颜色，合法值： r,g,b 或者 white,black,blue.	black
    kaptcha.obscurificator.impl	
        图片样式：<br />水纹 com.google.code.kaptcha.impl.WaterRipple <br />
        鱼眼 com.google.code.kaptcha.impl.FishEyeGimpy <br />
        阴影 com.google.code.kaptcha.impl.ShadowGimpy 
    com.google.code.kaptcha.impl.WaterRipple
    kaptcha.background.impl	背景实现类	com.google.code.kaptcha.impl.DefaultBackground
    kaptcha.background.clear.from	背景颜色渐变，开始颜色	light grey
    kaptcha.background.clear.to	背景颜色渐变， 结束颜色	white
    kaptcha.word.impl	文字渲染器	com.google.code.kaptcha.text.impl.DefaultWordRenderer
    kaptcha.session.key	session key	KAPTCHA_SESSION_KEY
    kaptcha.session.date	session date	KAPTCHA_SESSION_DATE
```

# 使用

## pom依赖
```xml
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

## bean注入
```xml
  <!-- 验证码 -->
    <bean id="captchaProducer" class="com.google.code.kaptcha.impl.DefaultKaptcha">
        <property name="config">
            <bean class="com.google.code.kaptcha.util.Config">
                <constructor-arg>
                    <props>
                        <!-- 这里的颜色只支持标准色和rgb颜色，不可使用十六进制的颜色 -->
                        <!-- 是否有边框 -->
                        <prop key="kaptcha.border">no</prop>
                        <!-- 验证码文本字符颜色 -->
                        <prop key="kaptcha.textproducer.font.color">black</prop>
                        <!-- 验证码图片宽度 -->
                        <prop key="kaptcha.image.width">92</prop>
                        <!-- 验证码图片高度 -->
                        <prop key="kaptcha.image.height">36</prop>
                        <!-- 验证码文本字符大小 -->
                        <prop key="kaptcha.textproducer.font.size">24</prop>
                        <!-- session中存放验证码的key键 -->
                        <prop key="kaptcha.session.key">code</prop>
                        <!-- 验证码噪点颜色 -->
                        <prop key="kaptcha.noise.color">white</prop>
                        <!-- 验证码文本字符间距 -->
                        <prop key="kaptcha.textproducer.char.space">3</prop>
                        <!-- 验证码样式引擎 -->
                        <prop key="kaptcha.obscurificator.impl">com.google.code.kaptcha.impl.ShadowGimpy</prop>
                        <!-- 验证码文本字符长度 -->
                        <prop key="kaptcha.textproducer.char.length">4</prop>
                        <!-- 验证码文本字体样式 -->
                        <prop key="kaptcha.textproducer.font.names">宋体,楷体,微软雅黑</prop>
                    </props>
                </constructor-arg>
            </bean>
        </property>
    </bean>
```

## java代码
```java
@Controller
public class KaptchaController {
    /**
     * 验证码工具
     */
    @Autowired
    DefaultKaptcha defaultKaptcha;

    @RequestMapping("/get")
    public void defaultKaptcha(HttpServletRequest request, HttpServletResponse response) throws Exception {
        byte[] captcha = null;
        ByteArrayOutputStream out = new ByteArrayOutputStream();

        try {
            // 将生成的验证码保存在session中
            String createText = defaultKaptcha.createText();
            request.getSession().setAttribute("rightCode", createText);
            BufferedImage bi = defaultKaptcha.createImage(createText);
            ImageIO.write(bi, "jpg", out);
        } catch (Exception e) {
            response.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        captcha = out.toByteArray();
        response.setHeader("Cache-Control", "no-store");
        response.setHeader("Pragma", "no-cache");
        response.setDateHeader("Expires", 0);
        response.setContentType("image/jpeg");
        ServletOutputStream sout = response.getOutputStream();
        sout.write(captcha);
        sout.flush();
        sout.close();
    }

    /**
     * 校对验证码
     *
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/check", method = RequestMethod.GET)
    public ResponseEntity check(HttpServletRequest request, HttpServletResponse response) {
        ModelAndView model = new ModelAndView();
        String rightCode = (String) request.getSession().getAttribute("rightCode");
        String tryCode = request.getParameter("tryCode");
        System.out.println("rightCode:" + rightCode + " ———— tryCode:" + tryCode);
        if (!rightCode.equals(tryCode)) {
            return ResponseEntity.ok(new AbstractMap.SimpleEntry(false, "验证码错误,请再输一次!"));
        } else {
            return ResponseEntity.ok(new AbstractMap.SimpleEntry(true, "ok"));
        }
    }
}

```