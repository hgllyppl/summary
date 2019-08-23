# 配置
## project defaults
        settings
            editor -> code style -> java -> imports -> class count/names count
            editor -> inspections -> duplicate code
            editor -> file encodings
            build -> build tools -> maven -> importing -> auto/sources/jdk
                                          -> runner -> jdk/skip tests
            other settings -> auto import -> add unambiguous imports on the fly
        project structure
            project -> sdk & language level
            SDKs

## preferences
        keymap
            -> settings(alt+s)
            -> edit configurations(alt+c)
            -> delete(ctrl+d)
            -> reformat code(alt+f)
            -> show diff(shelve/diff)(alt+d)
            -> optimize imports(alt+o)
            -> reveal in finder/show in explorer(alt+e)
            -> rename(alt+r)
            -> project structure(alt+p)
        editor -> general -> appearance -> show line numbers
                                        -> show parameter name hints
                          -> code completion -> case sensitive
               -> colors & fonts -> font -> consolas/14
                                 -> console font -> consolas/14
               -> live templates -> other -> avm/ivm/main/prsvm/prvm/pusvm/puvm/tryc/trycf/tryf/tvm
                                 -> plain -> prsf/pusf/st
               -> file types -> ignore files and folders -> .idea;*.iml;
        build -> debugger -> stepping -> skip class loaders
                                      -> skip simple getters
                                      -> do not step into the classes
                                            java.*
                                            javax.*
                                            junit.*
                                            org.slf4j.*
                                            org.apache.commons.logging.*
                                            java.util.logging.*
                                            org.apache.log4j.*
                                            ch.qos.logback.*
                                            org.springframework.boot.logging.logback.*
                                            org.springframework.boot.ansi.*
                          -> hotswap -> reload classes after compilation -> always

## generate
        toString -> settings -> templates
        ToStringBuilder(Fast-JSON)  
        public String toString() {
            return com.alibaba.fastjson.JSON.toJSONString(this);
        }

## live templates
- other
    ```java
    avm
        public abstract void $VAR$();
    ivm
        void $VAR$();
    main
        public static void main(String[] args){
            $END$
        }
    prsvm
        private static void $VAR$(){
            $END$
        }
    prvm
        private void $VAR$(){
            $END$
        }
    pusvm
        public static void $VAR$(){
            $END$
        }
    puvm
        public void $VAR$(){
            $END$
        }
    tryc
        try {
            $END$
        } catch ($VAR$ e) {

        }
    trycf
        try {
            $END$
        } catch ($VAR$ e) {

        } finally {

        }
    tryf
        try {
            $END$
        } finally {

        }
    tvm
        @org.junit.Test
        public void test_$var$(){
            $END$
        }
    ```
- plain

        prsf private static final
        pusf public static final
        st String

# FAQ
0. run/debug卡顿
<br>将机器名称、localhost映射到127.0.0.1
0. debug变慢
<br>取消方法断点
<br>取消方法返回值
