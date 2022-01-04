---
layout: post
title:  "Intro to Web scraping!"
date:   2021-09-08 20:30:00 -0800
categories: Technology
---

#### A beginner’s guide to web scraping in Node.js



### Preface
This article is an attempt to share what I know about web scraping and help the developer community. It is a way to express my thoughts, spread the knowledge and most importantly, help others! Till date, I do code and the code snippets are from my own working code :wink:

### Summary

>TL;DR If you are new to web scraping and want to learn more about it, continue reading further. It has some code snippets in Nodejs to show how it is done

<p> Data is critical for modern day businesses to understand their customers, competitors and to excel in the modern digital world. Companies with the right data strategy will be able to collect, cleanse and use the data that will help accelerate the business growth. </p>

![Web Scraping](/assets/webscraping.jpeg)
Image Credit: https://www.datasciencecentral.com/

Every company has access to data in different ways. Here are some of them to note …
- Data provided by the customers or end users
- Data shared by partners
- Data available through social media
- Data available on the websites (public or competitor sites) that is publicly accessible

In this article, I want to share some details about web scraping — an approach used to scrape the data that we want from the public internet pages. There are various languages, libraries and approaches in which you can perform scraping. In this page, I am going to use Node.js and some of the libraries.

### Start with some basics
- **Website Layout:** Understand the layout and structure of the web site that you want to scrape. Every website and every page needs different handling.
- **Data Elements:** Identify the data elements that you want to scrape — elements of each page as well as various pages of a website. Keep in mind, that many at times, the websites will redirect the pages too. You need some knowledge of CSS tags and Java script to get the needed information
- **Cookies** — yum! everyone loves it. Of course same is the case with websites too. When you visit any page, they leave cookies on your computer to track and understand you as well as to store information across sessions. We need to create cookies and send in our web scraping calls to make sure the companies doesn’t identify our program as Robots. Many eCommerce / internet companies know about web scraping and they try to block the efforts done my other engineers to scrape the sites.
- **Scraping volume and speed** — make sure you don’t bombard the websites at very high volumes so that they don’t flag your session and you might also accidentally bring their servers down if they are a small website.
- **Proxy providers** — When you do scraping at a small volume, you don’t need to use them, otherwise, we need to work with Proxy providers so that they rotate the IP addresses and mask our identity. That way, if some IP’s are blocked, we scrape from different IP’s

### Legal & Compliance
It is legal to scrape public information, but you are not allowed to scrape any data that is behind a login etc. Please check with your manager and legal teams before you scrape the data to make sure you and your company are in compliance with the rules set by the organization
_Disclaimer: This is based on my knowledge and it is not a legal advice_

### Buy (3rd party) vs Build (in-house solutions)
There are many 3rd parties that provide the scraping as a service for monthly fee. They are good for small volume, but when we want to do this at scale for thousands of items across many websites, the bill will start adding up. This is where in-house solutions over long term and high volumes will give us the best advantage. Also, this can just be your side project to show the org the power of scraping.

### Show me the Code
Exciting to learn? Here is a simple way to start scraping the data — like I said above, we will use Node.js for this.

##### _Pre-requisites_
- You need to have [Node.js](https://nodejs.dev/) installed on your machine
- We will use [Cheerio](https://github.com/cheeriojs/cheerio) for parsing HTML and XML in Node.js. Cheerio makes web scraping so easy and it is fast and flexible. There are many other alternatives too.
- We will use [axios](https://axios-http.com/docs/intro) package to fetch the markup from the websites to feed into Cheerio. Again, this is one among many others that you can use
- Optionally, Install [pretty](https://www.npmjs.com/package/pretty) for beautifying the markup when reading on terminal
- For cookie management, we will use [tough-cookie](https://www.npmjs.com/package/tough-cookie). We will install tough-cookie as well as [axios-cookiejar-support](https://www.npmjs.com/package/axios-cookiejar-support)
- For handling the files, we will use [fs](https://www.npmjs.com/package/fs) as well as [csvtojson converter](https://www.npmjs.com/package/csvtojson)
- Make sure your have a text editor available
- Basic understanding of JavaScript, Node.js, and the Document Object Model DOM. If not, there are many sites where you can learn this quickly

##### _Let’s get going_

For windows cmd or powershell or using GUI editors, you might need to tweak the commands a little. I am listing this for a Unix machine

1. **Create a working directory** — Choose any name you like
    ```mkdir <dirname>```
2. **Initialize the project and install the dependencies**

    Once you are in the working directory you have created above, initialize the project and install dependencies. This will create a `package.json` file in this root directory and will keep track of your installations and the versions.

    ```node
    pkg install nodejs
    npm init -y
    npm install cheerio axios (you can install multiple things like this or separately)
    npm install tough-cookie
    npm install axios-cookiejar-support
    npm install csv-parser
    npm install pretty
    ```

3. **Inspect the web page**

    Spend some time on the website that you want to scrape and identify the data elements that you are interested in. Understand the HTML structure. Open the dev tools on your browser using menu options or by pressing `CTRL+SHIFT+I`. You can also right click and select Inspect element.

    In this picture below, you can see how it is showing the respective tag for price that I am inspecting

    ![Inspect Tags](/assets/webscraping-ex1.png)

4. **Page URL**

    Figure out on how you can find the page you are looking for in the website. This changes from site to site. Also, there are some URL’s you can scrape and some are blocked. This is mentioned in Robots.txt of the respective web site.

    Page inspection and Page URL is where you spend most of your time on. This is the crux of the problem to find out pages, redirection, any loop holes etc. Here are some clues.
- When you see complex URL’s with the item number in the ending like `https://<website>/p/<something>/something>/12345` …. you can try a simple `https://<website>/p/12345` and you might get auto redirected
- Search URL’s are also your friends to land you in the right page … `https://<website>/find.html?find=12345` or `https://<website>/search/12345` .. look for these patterns

5. **Write the code**
- Start with your file. Call it as <whatevername>.js
- Load the dependencies
    ```node
    // Loading dependencies
    const axios = require(“axios”);
    const axiosCookieJarSupport = require(‘axios-cookiejar-support’).default;
    const tough = require(‘tough-cookie’);
    const cheerio = require(“cheerio”);
    const fs = require(“fs”);
    const csv = require(‘csvtojson’);
    ```
- Assuming that you are passing your items in a CSV file or pass the items as arguments, read the arguments
    ```
    // Reading Arguments
    const args = process.argv.slice(2);
    const fileName = args[0];
    ```

- Do the cookie management. We will use cookie jar here and also add in additional options as needed. This is critical is manipulating the websites that you are not a robot. You can also look at cookie on your laptop to understand more of this
    ```
    // Cookie Management
    axiosCookieJarSupport(axios);
    const cookieJar = new tough.CookieJar();
    cookieJar.setCookieSync(‘key=value; deviceType=is_desktop’, ‘<website domain>’);
    ```

- You can use the CSV to JSON dependency that we loaded to read your CSV file and convert into JSON and do line by line operation. One other very important thing you will see here is the delay interval I added. This example has a 5 seconds delay between each request to not bombard the site
    ```
    // Reading from CSV and loading into array
    csv()
    .fromFile(fileName)
    .then((inArray) => {
    inArray1 = JSON.stringify(inArray);
    inArray2 = JSON.parse(inArray1);
    var interval = 5000; // setting up time delay between each product scraping
    var promise = Promise.resolve();
    inArray2.forEach(function(inRow) {
    promise = promise.then(function() {
    scrapeData(inRow.ITEM); // Call scraping function for each Item in the file
    return new Promise(function(resolve) {
    setTimeout(resolve, interval);
    });
    });
    });
    })
    }
    ```

- Use axios to make the call and get the results and extract the content. While making the request set up necessary headers and pass on the cookie you sent above. Many times, you wont get the response unless you set this right

    In the below code, I am passing URL as a variable and using get, and once the response is received, I use “then” part of the code or “catch” part for error handling
    ```
    axios
    .get(url, {
    jar: cookieJar,
    withCredentials: true,
    headers: {
    accept: “text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9”,
    Cache: “no-cache”,
    connection: “keep-alive”,
    “User-Agent”: “Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36 Edg/92.0.902.84”,
    “accept-language”: “en-US,en;q=0.9”
    }
    })
    .then((response) => {
    // YOUR ELEMENT SCRAPING CODE here
    })
    .catch((err) => {
    console.error(err);
    });
    ```

- Here is a quick snippet of how you extract elements and how to use cheerio

    Read the response and load it into Cheerio
    ```
    const data = response.data; // get the main data from the response object
    const $ = cheerio.load(data); // Load HTML
    ```

Here is an example of CSS tags usage and taking the data
```const price_array = $(“input[type=’hidden’][id=’qubitEcProduct’]”).val();```

If you get a JSON string as a response or extract script, you can use JSON stringify and parse to read your elements
```
const resp = JSON.parse(price_array); // Parse the JSON
var inProductId = arguments[0];
var siteProductId = resp[0].product.productId;
```

Want to write to a file? No problem — let us do it. You can also write it to your databases
```
// Write data to the file
fs.writeFile(‘out.csv’, outData, {
flag: ‘a+’
}, err => {})
```

- Now sky is the limit. You can use sync or Async functions and get creative in what you are doing.

##### Execution
You can execute it by calling `node <whatevername>.js <inputfile.csv>`

### Conclusion
Enjoyed the article? Get going — grab a coffee, your laptop and a website and play with it. It is fun I promise!