viai18n-loader - another webpack loader i18n solution for Vue (Nuxt) with auto generated keys 
Currently under development, pull requests and suggestions are welcome.

# Why?
* We need auto generated key
    > There are only two hard things in Computer Science: cache invalidation and naming things. -- Phil Karlton
    
    Most i18n solutions ask developers to name each text with a unique key, like `$t("message.hello")`, it's not reasonable to waste time doing this job 
* Text translations should be put just beside source code
  
  Some solutions extract all texts to a single json file, it's not conform to modularization

# How it works?
1. Find automatically texts to translate by two means: regex string or 'separator'(Refer to usage below). 
2. Keep first 8 characters of every matched text as key (and add 4 characters md5 hash to the key if text is longer than 8)
3. Replace texts by translator markups using regex
4. Create or update `filename.messages.json` to store translations, import this file in `filename.vue`

# Usage

## Config webpack
Config example using nuxt.
**Given `regString`, the loader will match all texts/attributes/string templates by `new RegExp(regString)`
**Given `separator`, the loader will ignore `regString`, The loader will match all texts between `separator`, for example `##text##`.**
```
    {
        test: /\.vue$/,
        exclude: [/node_modules/],
        loader: 'vi18n-loader',
        enforce: 'pre',
        options: {
          updateMessagesFile: ctx.isClient && ctx.isDev, // only update messages file when it's dev and client(when using ssr)
          cacheTime: 3000,
          // regString to match simplified chinese characters
          regString: '[\u4e00-\u9fa5\u3002\uff1b\uff0c\uff1a\u2018\u2019\u201c\u201d\uff08\uff09\u3001\uff1f\uff01\ufe15\u300a\u300b]+',
          // separator: '##', // match texts surrounded by '##', like '##text##'
          // Loader will use existing translations, if there is not, will use text generated by translator
          languages: [{
            key: 'zh_Hans_CN',
            translator: (matched) => {
              // Delete repeat mark R, sometimes we need a text to be translated differently
              return matched.replace(/^[R]+/, '')
            }
          }, {
            key: 'zh_Hant_HK',
            translator: (matched) => {
              // example to auto translate simplified chinese to traditional
              return chineseS2T.s2t(matched.replace(/^[R]+/, ''))
            }
          }, {
            key: 'en_US',
            translator: (matched) => {
              return matched.replace(/^[R]+/, '')
            }
          }],
        }
      }
```
## In .vue file
* Add computed property `$lang` from which the plugin can read the current language.
**This `$lang` is necessary for the loader to put messages in the right place**
It can simply return the value from store, eg.:
```
    computed:{
        $lang(){
            return this.$store.getters.lang
        },
    }
```
Then you can easily change languages to show by mutate this property in the store, eg.:
```
    methods:{
        changeLang(lang){
            this.$store.dispatch('setLang', lang)
        }
    }
```

# Roadmap
- [ ] Add cli tool to group all `*.messages.json` and serve a web page for human translators
- [ ] Add example and better documentaion
- [ ] Add tests
