# 建造者模式

## 场景需求

一般用于构造复杂对象。

可以用于抽离复杂对象的构造函数，让我们可以通过多种方法的排列组合构建复杂对象。如果构造器参数过多，可以考虑`builder`模式。其实这是`builder`模式的一种变种，标准实现具体可参考GOF的相关阐述。

此处主要关注抽离复杂对象的构造函数问题。

## 方案分析

一般构造对象存在如下几种方案：

- 直接构造函数：直接通过构造函数传参的方式设置属性，这种方法如果属性过多的话会让构造函数十分臃肿。
    - 不能灵活的按需选择只设置某些参数
    - 需要我们手工确保参数顺序的准确性
    - 无法清晰地判断某个参数的具体含义

- 重叠构造器：其实就是构造函数的重载，先写第一个只有必要参数的构造器，第二个构造器有一个可选参数，第三个构造器有两个可选参数，以此类推；如果参数比较多时，类里面会出现一堆构造方法，并且阅读困难，很容易就把两个属性参数写颠倒位置了，编译不会出错，但运行就会出错了
    - 重载构造方法过多
    - 排列组合需求不够灵活

- JavaBean写法：首先生成Setter方法，然后生成初始对象，最后通过Setter方法按需给属性赋值，已经比较接近`builder`模式的思想了，且Setter方法也可以实现为链式方式。
    - 构造过程被分到几个调用中，在构造过程中JavaBean可能处于不一致的状态。类无法仅仅通过检验构造器参数的有效性来保证一致性。试图使用处于不一致状态的对象，将导致失败，这种失败与包含错误的代码大相径庭，因此它调试起来十分困难。
    - 另外JavaBean模式阻止了把类做成不可变的可能，这需要程序员付出格外的努力来确保它的线程安全。
> 引用一段解释：JavaBean方式是对象构造与字段赋值分离的，有可能线程1在给对象obj赋值，还没有赋值完成的时候，线程2就拿了obj的值了，此时就会发生不一致的情况；如果obj的字段全都是final的，不会出现上面那种情况，但字段只能会通过构造函数赋值（或者`builder`模式也行），而无法使用JavaBean的Setter函数赋值。 所以有多线程要求的，比如是传给消息队列的对象，程序员要自己保证线程安全。

- `builder`方式：既能保证像重叠构造器那样的安全，也能实现JavaBean模式那样的可读性。
    - 可以按需装配，无需考虑装配顺序
    - 通过方法可以知名达意理解属性含义
    - 安全性：`Builder.build()`方法之前都不会创建对象，所有的属性设置都必须在`Builder.build()`方法之前，同时，通过`Builder.build()`创建了对象后，其对象属性就不能改了，这也就保证了对象状态的一致性

## builder实现步骤
此处实现的是一个内部类构造器的方案。

- 按需final化对象属性
- 只生成Getter方法，注意：`待构造类中没有任何Setter方法`
- 私有化构造方法，参数为后面要讲到的Builder构造器
- 创建`public static final class`类型的内部类Builder类，属性与外部待构造类属性一致（但一般不能`final`，否则构造器自己也不能按需设置了）
- 内部类Builder类生成链式设置属性方法（一般属性名作为方法名），注意：`Builder类中没有任何Getter方法`
- 可按需在Builder中的相关Setter方法中验证属性的必输性及相关业务规则，也可将必输属性添加到Builder的构造函数中
- 按需创建newBuilder静态方法，可通过待构造类反向初始化一个Builder对象
- 如何简化创建过程：IDEA中使用`InnerBuilder`插件即可，生成时可勾选如下选项
    - Generate static newBuilder() method
    - Generate builder copy constructor
    - Use field names in setter
- `builder`如何使用：

```java
RequestData<String> requestData = RequestData.newBuilder()
                // Feign客户端: 微服务代码, 也即domainUrl字段
                .serviceName(interfaceServerDTO.getDomainUrl())
                .httpMethod(HttpMethod.valueOf(interfaceDTO.getRequestMethod()))
                .url(interfaceDTO.getInterfaceUrl())
                // 请求参数
                .addHeaders(requestPayloadDTO.getHeaderParamMap())
                .addPathVariables(requestPayloadDTO.getPathVariableMap())
                .addRequestParams(requestPayloadDTO.getRequestParamMap())
                .addMultipartFiles(requestPayloadDTO.getMultipartFileMap())
                .payload(requestPayloadDTO.getPayload())
                .build();
```

## 附源码
> 注意：此处为了便于Builder的操作，将相关Map属性在Builder中设置为了`final`，同时，在`InnerBuilder`生成的基础代码之上，重写了相关Setter方法为addXxx/addXxxs方法

```java
package org.hzero.integration.sdk.domain;

import java.util.HashMap;
import java.util.Map;

import org.apache.http.entity.ContentType;
import org.springframework.http.HttpMethod;
import org.springframework.web.multipart.MultipartFile;

/**
 * 请求参数
 *
 * @author allen.liu
 * @date 2019/5/31
 */
public class RequestData<T> {
    /**
     * 请求地址
     */
    private final String url;
    /**
     * 请求方法
     */
    private final HttpMethod httpMethod;
    /**
     * Reqeust Header参数列表
     */
    private final Map<String, String> headers;
    /**
     * Path变量列表
     */
    private final Map<String, String> pathVariables;
    /**
     * 请求查询参数列表
     */
    private final Map<String, String> requestParams;
    /**
     * Form-Data请求时文件参数
     */
    private final Map<String, MultipartFile> multipartFiles;
    /**
     * 指定payload实体内容
     */
    private final T payload;

    /**
     * 内容类型
     */
    private final ContentType contentType;

    /**
     * 认证参数
     */
    private final HttpAuthentication httpAuthentication;
    private final SoapAuthentication soapAuthentication;

    /**
     * SOAP参数
     */
    private final SoapData soapData;

    /**
     * ADD 20190816
     * 动态feign调用需要此字段
     * 服务名称
     */
    private final String serviceName;

    private RequestData(Builder<T> builder) {
        url = builder.url;
        httpMethod = builder.httpMethod;
        headers = builder.headers;
        pathVariables = builder.pathVariables;
        requestParams = builder.requestParams;
        multipartFiles = builder.multipartFiles;
        payload = builder.payload;
        contentType = builder.contentType;
        httpAuthentication = builder.httpAuthentication;
        soapAuthentication = builder.soapAuthentication;
        soapData = builder.soapData;
        serviceName = builder.serviceName;
    }

    public static Builder newBuilder() {
        return new Builder();
    }

    public static Builder newBuilder(RequestData requestData) {
        Builder builder = new Builder();

        builder.url = requestData.url;
        builder.httpMethod = requestData.httpMethod;
        builder.addHeaders(requestData.headers);
        builder.addPathVariables(requestData.pathVariables);
        builder.addRequestParams(requestData.requestParams);
        builder.addMultipartFiles(requestData.multipartFiles);
        builder.payload = requestData.payload;
        builder.contentType = requestData.contentType;
        builder.httpAuthentication = requestData.httpAuthentication;
        builder.soapAuthentication = requestData.soapAuthentication;
        builder.soapData = requestData.soapData;
        builder.serviceName = requestData.serviceName;

        return builder;
    }

    public String getUrl() {
        return url;
    }

    public HttpMethod getHttpMethod() {
        return httpMethod;
    }

    public Map<String, String> getHeaders() {
        return headers;
    }

    public Map<String, String> getPathVariables() {
        return pathVariables;
    }

    public Map<String, String> getRequestParams() {
        return requestParams;
    }

    public Map<String, MultipartFile> getMultipartFiles() {
        return multipartFiles;
    }

    public T getPayload() {
        return payload;
    }

    public ContentType getContentType() {
        return contentType;
    }

    public HttpAuthentication getHttpAuthentication() {
        return httpAuthentication;
    }

    public SoapAuthentication getSoapAuthentication() {
        return soapAuthentication;
    }

    public SoapData getSoapData() {
        return soapData;
    }

    public String getServiceName() {
        return serviceName;
    }

    public static final class Builder<T> {
        private String url;
        private HttpMethod httpMethod;
        private final Map<String, String> headers = new HashMap<>(16);
        private final Map<String, String> pathVariables = new HashMap<>(16);
        private final Map<String, String> requestParams = new HashMap<>(16);
        private final Map<String, MultipartFile> multipartFiles = new HashMap<>(8);
        private T payload;
        private ContentType contentType;
        private HttpAuthentication httpAuthentication;
        private SoapAuthentication soapAuthentication;
        private SoapData soapData;
        private String serviceName;

        private Builder() {
        }

        public Builder url(String url) {
            this.url = url;
            return this;
        }

        public Builder httpMethod(HttpMethod httpMethod) {
            this.httpMethod = httpMethod;
            return this;
        }

        public Builder addHeader(String key, String value) {
            this.headers.put(key, value);
            return this;
        }

        public Builder addHeaders(Map<String, String> headers) {
            if (headers != null) {
                this.headers.putAll(headers);
            }
            return this;
        }

        public Builder removeHeader(String key) {
            this.headers.remove(key);
            return this;
        }

        public Builder addPathVariable(String key, String value) {
            this.pathVariables.put(key, value);
            return this;
        }

        public Builder addPathVariables(Map<String, String> pathVariables) {
            if (pathVariables != null) {
                this.pathVariables.putAll(pathVariables);
            }
            return this;
        }

        public Builder addRequestParam(String key, String value) {
            this.requestParams.put(key, value);
            return this;
        }

        public Builder addRequestParams(Map<String, String> requestParams) {
            if (requestParams != null) {
                this.requestParams.putAll(requestParams);
            }
            return this;
        }

        public Builder addMultipartFile(String key, MultipartFile value) {
            this.multipartFiles.put(key, value);
            return this;
        }

        public Builder addMultipartFiles(Map<String, MultipartFile> multipartFiles) {
            if (multipartFiles != null) {
                this.multipartFiles.putAll(multipartFiles);
            }
            return this;
        }

        public Builder payload(T payload) {
            this.payload = payload;
            return this;
        }

        public Builder contentType(ContentType contentType) {
            this.contentType = contentType;
            return this;
        }

        public Builder httpAuthentication(HttpAuthentication httpAuthentication) {
            this.httpAuthentication = httpAuthentication;
            return this;
        }

        public Builder soapAuthentication(SoapAuthentication soapAuthentication) {
            this.soapAuthentication = soapAuthentication;
            return this;
        }

        public Builder soapData(SoapData soapData) {
            this.soapData = soapData;
            return this;
        }

        public Builder serviceName(String serviceName) {
            this.serviceName = serviceName;
            return this;
        }

        public RequestData build() {
            return new RequestData(this);
        }
    }
}
```

## 其他参考-Okhttp3.Request

- 源码

```java
/*
 * Copyright (C) 2013 Square, Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package okhttp3;

import java.net.URL;
import java.util.List;
import javax.annotation.Nullable;
import okhttp3.internal.Util;
import okhttp3.internal.http.HttpMethod;

/**
 * An HTTP request. Instances of this class are immutable if their {@link #body} is null or itself
 * immutable.
 */
public final class Request {
  final HttpUrl url;
  final String method;
  final Headers headers;
  final @Nullable RequestBody body;
  final Object tag;

  private volatile CacheControl cacheControl; // Lazily initialized.

  Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tag = builder.tag != null ? builder.tag : this;
  }

  public HttpUrl url() {
    return url;
  }

  public String method() {
    return method;
  }

  public Headers headers() {
    return headers;
  }

  public String header(String name) {
    return headers.get(name);
  }

  public List<String> headers(String name) {
    return headers.values(name);
  }

  public @Nullable RequestBody body() {
    return body;
  }

  public Object tag() {
    return tag;
  }

  public Builder newBuilder() {
    return new Builder(this);
  }

  /**
   * Returns the cache control directives for this response. This is never null, even if this
   * response contains no {@code Cache-Control} header.
   */
  public CacheControl cacheControl() {
    CacheControl result = cacheControl;
    return result != null ? result : (cacheControl = CacheControl.parse(headers));
  }

  public boolean isHttps() {
    return url.isHttps();
  }

  @Override public String toString() {
    return "Request{method="
        + method
        + ", url="
        + url
        + ", tag="
        + (tag != this ? tag : null)
        + '}';
  }

  public static class Builder {
    HttpUrl url;
    String method;
    Headers.Builder headers;
    RequestBody body;
    Object tag;

    public Builder() {
      this.method = "GET";
      this.headers = new Headers.Builder();
    }

    Builder(Request request) {
      this.url = request.url;
      this.method = request.method;
      this.body = request.body;
      this.tag = request.tag;
      this.headers = request.headers.newBuilder();
    }

    public Builder url(HttpUrl url) {
      if (url == null) throw new NullPointerException("url == null");
      this.url = url;
      return this;
    }

    /**
     * Sets the URL target of this request.
     *
     * @throws IllegalArgumentException if {@code url} is not a valid HTTP or HTTPS URL. Avoid this
     * exception by calling {@link HttpUrl#parse}; it returns null for invalid URLs.
     */
    public Builder url(String url) {
      if (url == null) throw new NullPointerException("url == null");

      // Silently replace web socket URLs with HTTP URLs.
      if (url.regionMatches(true, 0, "ws:", 0, 3)) {
        url = "http:" + url.substring(3);
      } else if (url.regionMatches(true, 0, "wss:", 0, 4)) {
        url = "https:" + url.substring(4);
      }

      HttpUrl parsed = HttpUrl.parse(url);
      if (parsed == null) throw new IllegalArgumentException("unexpected url: " + url);
      return url(parsed);
    }

    /**
     * Sets the URL target of this request.
     *
     * @throws IllegalArgumentException if the scheme of {@code url} is not {@code http} or {@code
     * https}.
     */
    public Builder url(URL url) {
      if (url == null) throw new NullPointerException("url == null");
      HttpUrl parsed = HttpUrl.get(url);
      if (parsed == null) throw new IllegalArgumentException("unexpected url: " + url);
      return url(parsed);
    }

    /**
     * Sets the header named {@code name} to {@code value}. If this request already has any headers
     * with that name, they are all replaced.
     */
    public Builder header(String name, String value) {
      headers.set(name, value);
      return this;
    }

    /**
     * Adds a header with {@code name} and {@code value}. Prefer this method for multiply-valued
     * headers like "Cookie".
     *
     * <p>Note that for some headers including {@code Content-Length} and {@code Content-Encoding},
     * OkHttp may replace {@code value} with a header derived from the request body.
     */
    public Builder addHeader(String name, String value) {
      headers.add(name, value);
      return this;
    }

    public Builder removeHeader(String name) {
      headers.removeAll(name);
      return this;
    }

    /** Removes all headers on this builder and adds {@code headers}. */
    public Builder headers(Headers headers) {
      this.headers = headers.newBuilder();
      return this;
    }

    /**
     * Sets this request's {@code Cache-Control} header, replacing any cache control headers already
     * present. If {@code cacheControl} doesn't define any directives, this clears this request's
     * cache-control headers.
     */
    public Builder cacheControl(CacheControl cacheControl) {
      String value = cacheControl.toString();
      if (value.isEmpty()) return removeHeader("Cache-Control");
      return header("Cache-Control", value);
    }

    public Builder get() {
      return method("GET", null);
    }

    public Builder head() {
      return method("HEAD", null);
    }

    public Builder post(RequestBody body) {
      return method("POST", body);
    }

    public Builder delete(@Nullable RequestBody body) {
      return method("DELETE", body);
    }

    public Builder delete() {
      return delete(Util.EMPTY_REQUEST);
    }

    public Builder put(RequestBody body) {
      return method("PUT", body);
    }

    public Builder patch(RequestBody body) {
      return method("PATCH", body);
    }

    public Builder method(String method, @Nullable RequestBody body) {
      if (method == null) throw new NullPointerException("method == null");
      if (method.length() == 0) throw new IllegalArgumentException("method.length() == 0");
      if (body != null && !HttpMethod.permitsRequestBody(method)) {
        throw new IllegalArgumentException("method " + method + " must not have a request body.");
      }
      if (body == null && HttpMethod.requiresRequestBody(method)) {
        throw new IllegalArgumentException("method " + method + " must have a request body.");
      }
      this.method = method;
      this.body = body;
      return this;
    }

    /**
     * Attaches {@code tag} to the request. It can be used later to cancel the request. If the tag
     * is unspecified or null, the request is canceled by using the request itself as the tag.
     */
    public Builder tag(Object tag) {
      this.tag = tag;
      return this;
    }

    public Request build() {
      if (url == null) throw new IllegalStateException("url == null");
      return new Request(this);
    }
  }
}
```

- 使用

```java
Request request = new Request.Builder()
                .url("http://www.weather.com.cn/xxxx")
                .addHeader("header","header")
                .put("RequestBody")
                .build();
```