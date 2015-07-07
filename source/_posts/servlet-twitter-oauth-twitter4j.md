Title: 在Servlet中使用Twitter OAuth认证
Date: 2010-10-31 22:09
Author: mcxiaoke
Category: Java
Tags: Twitter, Java, OAuth
Slug: servlet-twitter-oauth-twitter4j

例子来自Twitter4j的作者，我自己的代码等完善了再发布：

主要原理是在Twitter验证完毕后重定向到Callback的网址时，获取网址后面的oauth_verifier参数，进而获取AccessToken，并存储供以后使用。

SigninServlet

``` 
package twitter4j.examples.signin;

import twitter4j.Twitter;  
import twitter4j.TwitterException;  
import twitter4j.http.RequestToken;

import javax.servlet.ServletException;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;

public class SigninServlet extends HttpServlet {  
private static final long serialVersionUID = -6205814293093350242L;

protected void doGet(HttpServletRequest request, HttpServletResponse
response) throws ServletException, IOException {  
Twitter twitter = new Twitter();  
request.getSession().setAttribute("twitter", twitter);  
try {  
StringBuffer callbackURL = request.getRequestURL();  
int index = callbackURL.lastIndexOf("/");  
callbackURL.replace(index, callbackURL.length(),
"").append("/callback");

RequestToken requestToken =
twitter.getOAuthRequestToken(callbackURL.toString());  
request.getSession().setAttribute("requestToken", requestToken);  
response.sendRedirect(requestToken.getAuthenticationURL());

} catch (TwitterException e) {  
throw new ServletException(e);  
}

}  
}

```

CallbackServlet

```
package twitter4j.examples.signin;

import twitter4j.Twitter;  
import twitter4j.TwitterException;  
import twitter4j.http.RequestToken;

import javax.servlet.ServletException;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;

public class CallbackServlet extends HttpServlet {  
private static final long serialVersionUID = 1657390011452788111L;

protected void doGet(HttpServletRequest request, HttpServletResponse
response) throws ServletException, IOException {  
Twitter twitter = (Twitter)
request.getSession().getAttribute("twitter");  
RequestToken requestToken = (RequestToken)
request.getSession().getAttribute("requestToken");  
String verifier = request.getParameter("oauth\_verifier");  
try {  
twitter.getOAuthAccessToken(requestToken, verifier);  
request.getSession().removeAttribute("requestToken");  
} catch (TwitterException e) {  
throw new ServletException(e);  
}  
response.sendRedirect(request.getContextPath()+ "/");  
}  
}

```

