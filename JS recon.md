Description
--
This is a simple guide to perform javascript recon in the bugbounty


Steps
--

 - The first step is to collect possibly several javascript files (`more files` = `more paths,parameters` -> `more vulns`)
 
    To get more js files, this depends a lot on the target, I'm one who focuses a lot in large targets, it depends also a       lot on the tools that you use, I use a lot of my personal tools for this:
    
    __Tools:__
    
    
    gau  -  https://github.com/lc/gau  
    
    linkfinder -  https://github.com/GerbenJavado/LinkFinder
    
    getSrc - https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/getsrc.py 
    
    SecretFinder - https://github.com/m4ll0k/SecretFinder
    
    antiburl - https://github.com/tomnomnom/hacks/tree/master/anti-burl 
    
    antiburl.py - https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/antiburl.py
    
    ffuf - https://github.com/ffuf/ffuf
    
    allJsToJson.py (private tool)
    
    getJswords.py - https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/getjswords.py
    
    gitHubLinks.py (private tool)
    
    availableForPurchase.py - https://raw.githubusercontent.com/m4ll0k/Bug-Bounty-Toolz/master/availableForPurchase.py
    
    BurpSuite - http://portswigger.net/
    
    jsbeautify.py - https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/jsbeautify.py
    
    collector.py - https://github.com/m4ll0k/Bug-Bounty-Toolz/blob/master/collector.py
    
    getScriptTagContent.py (private tool)
    
    jsAlert.py (private tool)
    
    
  
     __Description:__
     
     __gau__ - This tool is great, i usually use it to search for as many javascript files as possible, many companies host                their files on third parties, this thing is very for important for a bughunter because then really enumerate                a lot js files! 
               
        Example:
              
        paypal.com host their files on paypalobjects.com
        
        $ gau paypalobjects.com |grep -iE '\.js'|grep -ivE '\.json'|sort -u  >> paypalJS.txt
        $ gau paypal.com |grep -iE '\.js'|grep -ivE '\.json'|sort -u  >> paypalJS.txt
        
        don't worry if where the files are hosted is out-of-scope, our intent is to enumerate js files to get more           
        parameters,paths,tokens,apikey,..
       
     __linkfinder__ - This tool is great, i usually use it to search paths,links, combined with `availableForPurchase.py` and `collector.py` is awesome!
      ```
      Example:
      
      $ cat paypalJS.txt|xargs -n2 -I@ bash -c "echo -e '\n[URL]: @\n'; python3 linkfinder.py -i @ -o cli" >> paypalJSPathsWithUrl.txt 
      $ cat paypalJSPathsWithUrl.txt|grep -iv '[URL]:'||sort -u > paypalJSPathsNoUrl.txt
      $ cat paypalJSPathsNoUrl.txt | python3 collector.py output
      ```
     __getSrc__ - Tool to extract script links, the nice thing about this tool it make absolute url!
   
        
         Example:
   
        $ python3 getSrc.py https://www.paypal.com/
   
        https://www.paypalobjects.com/digitalassets/c/website/js/react-16_6_3-bundle.js
        https://www.paypalobjects.com/tagmgmt/bs-chunk.js
      
      __SecretFinder__ - Tool to discover sensitive data like apikeys, accesstoken, authorizations, jwt,..etc in js file
      
      ```
      Example:
      
      $ cat paypalJS.txt|xargs -n2 -I @ bash -c 'echo -e "\n[URL] @\n";python3 linkfinder.py -i @ -o cli' >> paypalJsSecrets.txt
      
      ```
      __antiburl/antiburl.py__ - Takes URLs on stdin, prints them to stdout if they return a 200 OK. antiburl.py is an  advanced version
      
      ```
      Example:
      
      $ cat paypalJS.txt|antiburl > paypalJSAlive.txt
      $ cat paypalJS.txt | python3 antiburl.py -A -X 404 -H 'header:value' 'header2:value2' -N -C "mycookies=10" -T 50 
      
      ```
      
      __ffuf__ - tool for fuzzing, I also use it for fuzzing js files
      
      ```
     
      Example:
      
      $ ffuf -u https://www.paypalobjects.com/js/ -w jsWordlist.txt -t 200 
      
      Note: top wordlists - https://wordlists.assetnote.io/
      ```
      
     __allJsToJson.py__ - it makes a request to the urls that are passed to it and retrieves all the js files and saves them to me in a json file.
     ```js
     
     $ cat myPaypalUrls.txt | python3 allJsToJson.py output.json
     $ cat output.json
     
     {
    "url_1": {
        "root": "www.paypal.com",
        "path": "/us/home",
        "url": "https://www.paypa.com/us/home",
        "count_js": "4",
        "results": {
            "script_1": "https://www.paypalobjects.com/web/res/dc9/99e63da7c23f04e84d0e82bce06b5/js/config.js",
            "content": "function()/**/"
        }
    },
    "url_2": {}
    }
     ```
     __gitHubLinks.py__ - find new links on GitHub, in this case only javascript links
   
     ```
      Example:
   
      $ python3 gitHubLinks.py www.paypalobjects.com|grep -iE '\.js'
      ```
     
     __availableForPurchase.py__ - this tools search if a domain is available to be purchase, this tool combined with linkfinder and collector is really powerful. Many times the developers for distraction mistake to write the domain, maybe the domain is importing an external javascript file ,...etc
     
     ```
     Example: 
     
     $ cat paypalJS.txt|xargs -I @ bash -c 'python3 linkfinder.py -i @ -o cli' | python3 collector.py output
     $ cat output/urls.txt | python3 availableForPurchase.py
     [NO]  www.googleapis.com 
     [YES] www.gooogleapis.com
     ```
    
    __BurpSuite__ - extract the content between the script tags, I usually use `getScriptTagContent.py`
    
    ![burp](https://i.imgur.com/8N3AOWF.png)
    
    after this save the content and use linkfinder 
    
    `$ python3 linkfinder.py -i burpscriptscontent.txt -o cli`
    
    
    __jsbeautify.py__ - Javascript Beautify 
    
    ```
    Example:
    
    $ python3 jsbeautify https://www.paypalobject.com/test.js paypal/manualAnalyzis.js
    
    ```
    
    __collector.py__ -  Split linkfinder stdout in jsfile,urls,params..etc
    
     ```
     $ python3 linkfinder.py -i https://www.test.com/a.js -o cli | python3 collector.py output
     $ ls output
     
     files.txt	js.txt		params.txt	paths.txt	urls.txt
     ```
     
    __jsAlert.py__ - notify if there are any interesting keywords, such as postMessage,onmessage,innerHTML,etc
    
    ```
    Example:
    
    $ cat myjslist.txt | python3 jsAlert.py
    
    [URL] https://..../test.js
    
    line:16 - innerHTML
    
    [URL] https://.../test1.js
    
    line:3223 - onmessage
    
    ```
     
    __getScriptTagContent.py__ - get content between script tags 
    
    ```
    Example:
    
    $ cat "https://www.google.com/"|python3 getScriptTagContent.py 
    
    function()/**/...
    ```
    
    __getJSWords.py__  - get all javascript file words excluding javascripts keywords
    
    ```
    Example:
    
    $ python3 getjswords.py https://www.google.com/test.js
    
    word
    word1
    ...
    ```
    
    As you see above we need a lot to do every time many requests, i solve this problem with allJsToJson, that keep me a contentof all js files and their content, obviously the tool is made on purpose to process only 5 urls at a time because of the size of the file, every time it process 5 urls save the output .. output1.json, output2.json,...
    

__Other Resources:__

- https://bhattsameer.github.io/2021/01/01/client-side-encryption-bypass-part-1.html
- https://developers.google.com/web/tools/chrome-devtools/javascript 
- https://www.youtube.com/watch?v=FTeE3OrTNoA&ab_channel=HackerOne

     
