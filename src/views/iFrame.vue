<template>
  <div class="iframe">
    <b-loading :is-full-page="false" :active.sync="isLoading" :can-cancel="true">
    </b-loading>
    <div class="label-selector" >
      <div
        class="_button"
        :class="{ '__active': selectedLabel === 'image' }"
        @click="SelectLabel('image')"
      >Image</div>
      <div
        class="_button"
        :class="{ '__active': selectedLabel === 'name' }"
        @click="SelectLabel('name')"
      >Name</div>
      <div
        class="_button"
        :class="{ '__active': selectedLabel === 'price' }"
        @click="SelectLabel('price')"
      >Price</div>
    </div>
    <iframe ref="iframe" id="frame"></iframe>

    <div class="selection">
      <div>GROUP</div>
      <div>LABEL</div>
      <div>TYPE</div>
      <div>LIST</div>
    </div>
    <!-- <vue-i-frame
            ref="frame"
            class="vue-i-frame"
            :src="pageLoader"
            @load="onLoad"
            @document-load="onDocumentLoad">
    </vue-i-frame>-->
  </div>
</template>

<script>
import VueIFrame from "vue-friendly-iframe";
import EventBus from "../EventBus";
import axios from "axios";

export default {
  data() {
    return {
      isLoading: true,
      classObject: {
        selectLabel: {
          selected: ""
        }
      },
      selectedLabel: "",
      selection: {},
      // externalPage: 'https://www.jd.co.th/product/sandisk-sdcz50c-016g-b35ge-flashdrive-16gb-usb2-0-green_2382816.html',
      externalPage: "https://www.jd.co.th/sale/pc/xvBycH1saJ.html",
      // pageLoader: 'about:blank',
      // pageLoader: 'https://www.jd.co.th/product/sandisk-sdcz50c-016g-b35ge-flashdrive-16gb-usb2-0-green_2382816.html',
      pageLoader: "https://www.jd.co.th/sale/pc/xvBycH1saJ.html"
    };
  },
  components: {
    VueIFrame
  },
  methods: {
    async Go(externalPage) {
      if (externalPage) {
        let pointer = { type: "", value: "" };
        this.pageLoader = externalPage;
        // axios.defaults.headers.get['Access-Control-Allow-Origin'] = '*';
        const page = axios
          .get(
            "http://bakatest.me:1234/api/v1/crawler/preview?url=" +
              externalPage
          )
          .then(r => {
            // console.log(r);
            const _this = this;
            const iframe = 
              this.$refs.iframe.contentDocument ||  this.$refs.iframe.contentWindow.document;
            EventBus.$emit('IFRAME_LOADED', { error: null, loading: true, loaded: false });
  
            this.$refs.iframe.onload = function() {
              _this.isLoading = false;
              EventBus.$emit('IFRAME_LOADED', { error: null, loading: false, loaded: true });
            }
  
            // CSS Injection
            const CSSStyleTag = document.createElement("style");
            CSSStyleTag.innerHTML =
              ".highlight {" + "color: purple;" + "background-color: red;";
            ("}");
  
            const injectCSS = `
                  <style>
                  .highlight {
                      color: purple;
                      background-color: red;
                  }
                  </style>
                  `;
  
            // const CSSLinkTag = document.createElement("link")
            // CSSLinkTag.rel = "stylesheet";
            // CSSLinkTag.type = "text/css";
            // console.log()
  
            console.log(iframe);
            iframe.open();
            iframe.write(r.data);
            iframe.write(injectCSS);
            // iframe.body.appendChild(CSSStyleTag);
            iframe.close();
            // iframe.innerHTML = r.data;
            // this.$refs.iframe.addEventListener('mouseover', this.elementHandler);
  
            (function() {
              var prevMouseover;
              var prev;
  
              if (iframe.addEventListener) {
                iframe.addEventListener("mouseover", mouseoverHandler, false);
                iframe.addEventListener("click", clickHandler, false);
              } else if (iframe.attachEvent) {
                iframe.attachEvent("mouseover", function(e) {
                  return handler(e || window.event);
                });
              } else {
                iframe.onmouseover = handler;
              }
  
              function mouseoverHandler(event) {
                console.log(event);
                const { path, target } = event;
                const { nodeName, lastChild, textContent } = target;
  
                if (
                  event.target === iframe ||
                  (prevMouseover && prevMouseover === event.target)
                ) {
                  return;
                }
                const enabledNodeLists = ["SPAN", "DIV", "LI", "P"]
                if (enabledNodeLists.includes(nodeName)) {
                  if (textContent !== "" || textContent !== null) {
                    console.log(textContent);
                    if (prevMouseover) {
                      // prev.className = prev.className.replace(/\bhighlight\b/, '');
                      prevMouseover.style.border = "";
                      prevMouseover = undefined;
                    }
        
                    if (event.target) {
                      event.preventDefault();
                      prevMouseover = event.target;
                      // prev.className += " highlight";
                      prevMouseover.style.border = "1spx solid red";
                    }
                    EventBus.$emit('IFRAME_ELEMENT_POINTING', { type: nodeName, value: textContent });
                  }
                  if (nodeName === "DIV") {
                    if (
                      lastChild === "text" ||
                      (nodeValue || nodeValue !== "" || nodeValue !== null || nodeValue !== "undefined") ||
                      (textContent !== "")
                    ) {
                      if (prevMouseover) {
                        // prev.className = prev.className.replace(/\bhighlight\b/, '');
                        prevMouseover.style.border = "";
                        prevMouseover = undefined;
                      }
        
                      if (event.target) {
                        event.preventDefault();
                        prevMouseover = event.target;
                        // prev.className += " highlight";
                        prevMouseover.style.border = "1px solid red";
                      }
                    }
                  }
                }
              }
  
              function clickHandler(event) {
                event.preventDefault();
                console.log(event.target);
                if (event.target.nodeName)
                // setAttribute('aria-disabled', true);
                console.log("CLICKED");
                if (event.target === iframe || (prev && prev === event.target)) {
                  console.log("ONE");
                  return;
                }
  
                if (prev) {
                  // prev.className = prev.className.replace(/\bhighlight\b/, '');
  
                  console.log("TWO");
                  prev.style.border = "";
                  prev = undefined;
                }
  
                if (event.target) {
                  console.log("THREE");
                  prev = event.target;
                  prev.className += " highlight";
                  console.log(event.target);
                  
                  EventBus.$emit('IFRAME_ELEMENT_POINTING', { type: nodeName, value: textContent, path: getPathTo(target) });

                  // prev.style.border = "1px solid green";
                  // let label = document.createElement("div");
                  // label.appendChild(document.createTextNode(_this.selectedLabel));
                  // label.className += "element-label";
                  // label.style.border = "1px solid green";
                  // prev.appendChild(label);
                }
              }
            })();
          })
          .catch(e => {
            console.log(e);
          });
      } else {
        console.log("URL Error");
        EventBus.$emit('IFRAME_LOADED', { error: true, loading: false, loaded: false });
      }
    },
    onLoad() {
      const vm = this;
      this.$refs.frame._vnode.elm.childNodes[0].id = "frame";
      const iframeContainer = document.getElementById("frame");
      // this.bypassXSSSecurity(iframe);
      const iframe = iframeContainer.contentDocument
        ? iframeContainer.contentDocument
        : iframeContainer.contentWindow.document;
      const iframeBody = iframe.body;
      

      // console.log(iframeBody);
      // const iframeScripts = iframeBody.querySelectorAll("script");
      // const iframeDivs = iframe.querySelectorAll("body");
      // for (let script of iframeScripts) {
      //     iframeScripts[script] = '';
      // }
      // console.log(iframeDivs);
      // for (let div of iframeDivs) {
      //     console.log(div);
      // }
      // iframeBody.addEventListener('mouseover', handler);
      // const allFrameElms = frameDoc.querySelectorAll("div");
      // console.log(allFrameElms);
      // this.loadHTML(document, iframeContainer);

      // (function() {
      //     var prev;

      //     if (iframeBody.addEventListener) {
      //         console.log("do 1");
      //         iframeBody.addEventListener('mouseover', handler);
      //     }
      //     else if (iframeBody.attachEvent) {
      //         console.log("do 2");
      //         iframeBody.attachEvent('mouseover', function(e) {
      //         return handler(e || window.event);
      //         });
      //     }
      //     else {
      //         console.log("do 3");
      //         iframeBody.onmouseover = handler;
      //     }

      // })();

      // function handler(event) {
      //     // console.log(event);
      //     console.log("handle")
      //     if (event.target === iframeBody ||
      //         (prev && prev === event.target)) {
      //     return;
      //     }
      //     if (prev) {
      //         prev.className = prev.className.replace(/\bhighlight\b/, '');
      //         prev.style.border = "";
      //         prev = undefined;
      //     }
      //     if (event.target) {
      //         prev = event.target;
      //         prev.style.border = "1px solid red";
      //     }
      // }
    },
    onDocumentLoad() {
      // // console.log(this.$refs.frame);
      // // console.log(this.$refs.frame._vnode.elm.id);
      // this.$refs.frame._vnode.elm.childNodes[0].id = "frame";
      // // this.$refs.frame._vnode.elm.attr('id','something');
      // // const frameDoc = this.$refs.frame._vnode.elm.children[0].contentDocument;
      // const frameDoc = document.querySelector("#frame");
      // console.log(frameDoc.contentDocument);
      // const allFrameElms = frameDoc.querySelectorAll("div");
      // console.log(allFrameElms);
      // // for (var i = 0; i < allFrameElms.length; i++) {
      // //     allFrameElms[i].addEventListener("click", function() {
      // //         console.log("clicked");
      // //     });
      // // }
    },
    SelectLabel(label) {
      this.selectedLabel = label;
    },
    bypassXSSSecurity(iframe) {
      let url = iframe.src;
      console.log("iFrameURL", url);
      let script = document.createElement("script");
      script.src =
        "https://query.yahooapis.com/v1/public/yql?q=select%20*%20from%20data.headers%20where%20url%3D%22" +
        encodeURIComponent(url) +
        "%22&format=json&diagnostics=true&env=store%3A%2F%2Fdatatables.org%2Falltableswithkey";
      document.body.appendChild(script);

      function getData(data) {
        console.log(this);
        if (
          data &&
          data.query &&
          data.query.results &&
          data.query.results.resources &&
          data.query.results.resources.content &&
          data.query.results.resources.status == 200
        )
          loadHTML(data.query.results.resources.content);
        else if (data && data.error && data.error.description)
          loadHTML(data.error.description);
        else loadHTML("Error: Cannot load " + url);
      }

      function loadHTML(html, iframe) {
        iframe.src = "about:blank";
        iframe.contentWindow.document.open();
        iframe.contentWindow.document.write(
          html.replace(
            /<head>/i,
            '<head><base href="' +
              url +
              '"><scr' +
              'ipt>document.addEventListener("click", function(e) { if(e.target && e.target.nodeName == "A") { e.preventDefault(); parent.loadURL(e.target.href); } });</scr' +
              "ipt>"
          )
        );
        iframe.contentWindow.document.close();
      }
    },
    loadHTML(html, iframe) {
      console.log(html);
      console.log(iframe);
      const url = iframe.src;
      // iframe.src = 'about:blank';
      iframe.contentWindow.document.open();
      iframe.contentWindow.document.write(
        html.replace(
          /<head>/i,
          '<head><base href="' +
            url +
            '"><scr' +
            'ipt>document.addEventListener("click", function(e) { if(e.target && e.target.nodeName == "A") { e.preventDefault(); parent.loadURL(e.target.href); } });</scr' +
            "ipt>"
        )
      );
      iframe.contentWindow.document.close();
    }
    // elementHandler(event) {
    //     // console.log(event);
    //     let prev;
    //     console.log("handle")
    //     if (event.target === document.body ||
    //         (prev && prev === event.target)) {
    //         return;
    //     }
    //     if (prev) {
    //         console.log(prev);
    //         // prev.className = prev.className.replace(/\bhighlight\b/, '');
    //         prev.style.border = "";
    //         prev = undefined;
    //     }
    //     if (event.target) {
    //         prev = event.target;
    //         prev.style.border = "1px solid red";
    //     }
    // }
  }
};

function getPathTo(element) {
    if (element.id!=='')
        return 'id("'+element.id+'")';
    if (element===document.body)
        return element.tagName;

    var ix= 0;
    var siblings= element.parentNode.childNodes;
    for (var i= 0; i<siblings.length; i++) {
        var sibling= siblings[i];
        if (sibling===element)
            return getPathTo(element.parentNode)+'/'+element.tagName+'['+(ix+1)+']';
        if (sibling.nodeType===1 && sibling.tagName===element.tagName)
            ix++;
    }
}

</script>

<style lang="scss">
.vue-i-frame,
#frame {
  width: 100%;
  min-height: 600px;
  height: 100%;
  display: block;
}

iframe {
  width: 100%;
  min-height: 600px;
  height: 100%;
  display: block;
}

.label-selector {
  display: flex;
  margin: auto;
  width: 600px;
  justify-content: center;

  & ._button {
    color: white;
    font-size: 0.8rem;
    text-transform: upper;
    line-height: 24px;
    border-radius: 24px;
    min-width: 100px;
    width: max-content;
    height: 24px;
    padding: 2px 4px;
    margin: 4px;
    background: black;

    &.__active {
      color: black;
      background: red;
    }
  }
}

.highlight {
  border: 1px solid green;

  & .element-label {
    color: white;
    font-size: 11px;
    text-transform: upper;
    line-height: 16px;
    border-radius: 12px;
    min-width: 44px;
    width: -webkit-max-content;
    width: -moz-max-content;
    width: max-content;
    height: 16px;
    padding: 2px 2px;
    margin: 0px;
    background: green;
    text-align: center;
  }
}

.iframe-loading-cover {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background: #0000005e;
}

.iframe {
  position: relative;
}
</style>
