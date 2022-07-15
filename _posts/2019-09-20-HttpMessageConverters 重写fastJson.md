---
layout: post
title: SpringBoot 中重写 HttpMessageConverters
categories: [HttpMessageConverters]
description: SpringBoot 中重写 HttpMessageConverters
keywords: HttpMessageConverters
tags: HttpMessageConverters
---


#### 重写

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.IOUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpInputMessage;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.MediaType;
import org.springframework.http.converter.AbstractHttpMessageConverter;
import org.springframework.http.converter.GenericHttpMessageConverter;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.http.converter.HttpMessageNotWritableException;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.lang.reflect.Type;
@Slf4j
public class MyHttpMessageConverter extends AbstractHttpMessageConverter<Object> implements GenericHttpMessageConverter<Object> {
    private FastJsonConfig fastJsonConfig = new FastJsonConfig();

    public MyHttpMessageConverter() {
        super(MediaType.ALL);
    }

    public boolean canRead(Type type, Class<?> contextClass, MediaType mediaType) {
        return super.canRead(contextClass, mediaType);
    }

    public boolean canWrite(Type type, Class<?> contextClass, MediaType mediaType) {
        return super.canWrite(contextClass, mediaType);
    }

    @Override
    public Object read(Type type, Class<?> contextClass, HttpInputMessage inputMessage)
            throws IOException, HttpMessageNotReadableException {

        InputStream in     = inputMessage.getBody();
        String      result = IOUtils.toString(in, "UTF-8");
        try {
            Object parseObject = JSON.parseObject(result, type, fastJsonConfig.getFeatures());
            log.info("[请求]-{}", result);
            return parseObject;
        } catch (Exception e) {
            log.info("[请求]-{}", result);
            throw e;
        }
    }

    @Override
    public void write(Object t, Type type, MediaType contentType, HttpOutputMessage outputMessage)
            throws IOException, HttpMessageNotWritableException {
        HttpHeaders headers = outputMessage.getHeaders();
        if (headers.getContentType() == null) {
            if (contentType == null || contentType.isWildcardType()
                    || contentType.isWildcardSubtype()) {
                contentType = getDefaultContentType(t);
            }
            if (contentType != null) {
                headers.setContentType(contentType);
            }
        }
        if (headers.getContentLength() == -1) {
            Long contentLength = getContentLength(t, headers.getContentType());
            if (contentLength != null) {
                headers.setContentLength(contentLength);
            }
        }
        writeInternal(t, outputMessage);
        outputMessage.getBody().flush();
    }

    @Override
    protected boolean supports(Class<?> clazz) {
        return true;
    }

    @Override
    protected Object readInternal(Class<? extends Object> clazz, HttpInputMessage inputMessage)
            throws IOException, HttpMessageNotReadableException {
        InputStream in = inputMessage.getBody();
        return JSON.parseObject(in, fastJsonConfig.getCharset(), clazz,
                fastJsonConfig.getFeatures());
    }

    @Override
    protected void writeInternal(Object obj, HttpOutputMessage outputMessage)
            throws IOException, HttpMessageNotWritableException {
        HttpHeaders           headers = outputMessage.getHeaders();
        ByteArrayOutputStream outnew  = new ByteArrayOutputStream();

        boolean writeAsToString = false;
        if (obj != null) {
            String className = obj.getClass().getName();
            if ("com.fasterxml.jackson.databind.node.ObjectNode".equals(className)) {
                writeAsToString = true;
            }
        }
        log.info("[响应]-{}", JSON.toJSONString(obj));
        if (writeAsToString) {
            String       text = obj.toString();
            OutputStream out  = outputMessage.getBody();
            out.write(text.getBytes());
            if (fastJsonConfig.isWriteContentLength()) {
                headers.setContentLength(text.length());
            }
        } else {
            int len = JSON.writeJSONString(outnew, //
                    fastJsonConfig.getCharset(), //
                    obj, //
                    fastJsonConfig.getSerializeConfig(), //
                    fastJsonConfig.getSerializeFilters(), //
                    fastJsonConfig.getDateFormat(), //
                    JSON.DEFAULT_GENERATE_FEATURE, //
                    fastJsonConfig.getSerializerFeatures());
            if (fastJsonConfig.isWriteContentLength()) {
                headers.setContentLength(len);
            }

            OutputStream out = outputMessage.getBody();
            outnew.writeTo(out);
        }

        outnew.close();

    }

    public FastJsonConfig getFastJsonConfig() {
        return fastJsonConfig;
    }

    public void setFastJsonConfig(FastJsonConfig fastJsonConfig) {
        this.fastJsonConfig = fastJsonConfig;
    }

}

```

### 使用fastJson做转bean处理

#### 配置 HttpMessageConverters
```java
    @Bean
    public HttpMessageConverters MyHttpMessageConverters() {
        MyHttpMessageConverter fastConverter  = new MyHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteDateUseDateFormat);
        fastConverter.setFastJsonConfig(fastJsonConfig);

        

        HttpMessageConverter<?> converter = fastConverter;
        return new HttpMessageConverters(converter);
    }
```


### feign 做外部调用在使用

feign做外部调用的时候，发现也会使用这个 Converters 进行读取与写出，在这个时候在高版本的时候会报类似的错误
```java
feign.codec.EncodeException: 'Content-Type' cannot contain wildcard type '*' 
```

#### 解决
```java
    @Bean
    public HttpMessageConverters MyHttpMessageConverters() {
        MyHttpMessageConverter fastConverter  = new MyHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteDateUseDateFormat);
        fastConverter.setFastJsonConfig(fastJsonConfig);

        List<MediaType> supportedMediaTypes = new ArrayList<>();
        supportedMediaTypes.add(MediaType.TEXT_HTML);
        supportedMediaTypes.add(MediaType.APPLICATION_JSON);
        supportedMediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
        //可以接着加
        supportedMediaTypes.add(new MediaType("text", "html", Charset.defaultCharset()));
        fastConverter.setSupportedMediaTypes(supportedMediaTypes);

        HttpMessageConverter<?> converter = fastConverter;
        return new HttpMessageConverters(converter);
    }

```
这样就可以了

也可以用如下的方式
即在每一个上面增加 consumes 参数

```java
@FeignClient("client")
public interface FocusService{
    /**
     * @param channel
     * @param page
     * @param userId
     * @return
     * @throws Exception
     */
    @RequestMapping(value="/crowd/v1/focus", method=RequestMethod.POST,consumes = MediaType.APPLICATION_JSON_VALUE)
    public ApiResponse<Focus> create (@RequestBody FocusCreate focusCondition);
```

### 参考
```
https://www.jianshu.com/p/1312b2b96858
https://www.cnblogs.com/xiaopotian/p/8654993.html
https://blog.csdn.net/lppl010_/article/details/94215233
```