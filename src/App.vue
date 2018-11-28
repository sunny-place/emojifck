<template lang="pug">
  #app
    button(@click="exec") RUN
    //-.bl_txtWrap
      label.bl_txt
        span >
        input(v-model="cmd.next")
        button(@click="addChr(cmd.next)") ↓
      label.bl_txt
        span <
        input(v-model="cmd.prev")
        button(@click="addChr(cmd.prev)") ↓
      label.bl_txt
        span +
        input(v-model="cmd.inc")
        button(@click="addChr(cmd.inc)") ↓
      label.bl_txt
        span -
        input(v-model="cmd.dec")
        button(@click="addChr(cmd.dec)") ↓
      label.bl_txt
        span [
        input(v-model="cmd.begin")
        button(@click="addChr(cmd.begin)") ↓
      label.bl_txt
        span ]
        input(v-model="cmd.end")
        button(@click="addChr(cmd.end)") ↓
      label.bl_txt
        span .
        input(v-model="cmd.write")
        button(@click="addChr(cmd.write)") ↓
      label.bl_txt
        span ,
        input(v-model="cmd.read")
        button(@click="addChr(cmd.read)") ↓
    .bl_txtWrap
      label.bl_txt
        span CODE
        textarea(v-model="code" @keydown="setCodePos" @click="setCodePos")
      label.bl_txt
        span INPUT
        textarea(v-model="input")
      label.bl_txt
        span RESULT
        textarea(v-model="result")
    
</template>

<script>

var charCode = [
"<NUL>","<SOH>","<STX>","<ETX>","<EOT>","<ENQ>","<ACK>","<BEL>","<BS>" ,"<HT>" ,
"<LF>" ,"<VT>" ,"<FF>" ,"<CR>" ,"<SO>" ,"<SI>" ,"<DLE>","<DC1>","<DC2>","<DC3>",
"<DC4>","<NAK>","<SYN>","<ETB>","<CAN>","<EM>" ,"<SUB>","<ESC>","<FS>" ,"<GS>" ,
"<RS>" ,"<US>" ,"<SPC>",  "!"  , "\""  ,  "#"  ,  "$"  ,  "%"  ,  "&"  ,  "'"  ,
  "("  ,  ")"  ,  "*"  ,  "+"  ,  ","  ,  "-"  ,  "."  ,  "/"  ,  "0"  ,  "1"  ,
  "2"  ,  "3"  ,  "4"  ,  "5"  ,  "6"  ,  "7"  ,  "8"  ,  "9"  ,  ":"  ,  ";"  ,
  "<"  ,  "="  ,  ">"  ,  "?"  ,  "@"  ,  "A"  ,  "B"  ,  "C"  ,  "D"  ,  "E"  ,
  "F"  ,  "G"  ,  "H"  ,  "I"  ,  "J"  ,  "K"  ,  "L"  ,  "M"  ,  "N"  ,  "O"  ,
  "P"  ,  "Q"  ,  "R"  ,  "S"  ,  "T"  ,  "U"  ,  "V"  ,  "W"  ,  "X"  ,  "Y"  ,
  "Z"  ,  "["  ,  "\\"  ,  "]"  ,  "^"  ,  "_"  ,  "`"  ,  "a"  ,  "b"  ,  "c"  ,
  "d"  ,  "e"  ,  "f"  ,  "g"  ,  "h"  ,  "i"  ,  "j"  ,  "k"  ,  "l"  ,  "m"  ,
  "n"  ,  "o"  ,  "p"  ,  "q"  ,  "r"  ,  "s"  ,  "t"  ,  "u"  ,  "v"  ,  "w"  ,
  "x"  ,  "y"  ,  "z"  ,  "{"  ,  "|"  ,  "}"  ,  "~"  ,"<DEL>"
];

class BrainFxck {
  constructor(){
    this.stack  = [];
    this.memLen = 5;
    this.memory = new Array(this.memLen);
    this.code = [];//[">","+","+","<","+","+","+","+",".",">","."];
    this.bfr_in  = [];
    this.bfr_out = "";
    this.mem_ptr = 0;
    this.cd_ptr = 0;
    this.bfr_ptr = 0;
    this.chr = charCode;
  }
  init() {
    this.stack  = [];
    this.memory = new Array(this.memLen);
    this.cmd = [];
    this.code = [];
    this.bfr_in  = [];
    this.bfr_out = "";
    this.mem_ptr = 0;
    this.cd_ptr = 0;
    this.bfr_ptr = 0;
  }
  execCode(code, input, cmd) {
    this.init();
    this.cmd  = cmd;
    this.code = this.genCode(code);
    this.bfr_in = input.split("");
    try {
      while(this.cd_ptr < this.code.length) {
        this.doCmd(this.code[this.cd_ptr]);
      }
    } catch(e) {
      this.bfr_out=e;        
    }
    return this.bfr_out;
  }
  doCmd(cmd) {
    switch(cmd) {
      case this.cmd.next:
        this.mem_ptr += 1;
        if(this.mem_ptr >= this.memLen){throw this.errorMsg("MEMORY_OVER")}
        this.cd_ptr += 1;
        break;
      case this.cmd.prev:
        this.mem_ptr -= 1;
        if(this.mem_ptr < 0){throw this.errorMsg("MEMORY_OVER")}
        this.cd_ptr += 1;
        break;
      case this.cmd.inc:
        this.memVal(1);
        this.cd_ptr += 1;
        break;
      case this.cmd.dec:
        this.memVal(-1);
        this.cd_ptr += 1;
        break;
      case this.cmd.begin:
        if(this.memVal() === 0) {
          var tmp = this.cd_ptr;
          do {
            this.cd_ptr += 1;
            console.log(this.cd_ptr + "--" + this.code.length);
            if(this.cd_ptr >= this.code.length) {this.cd_ptr = tmp; throw this.errorMsg("MISSING_END_BRACKET")}
          } while(this.code[this.cd_ptr] !== ']');
        } else {
          this.stack.push(this.cd_ptr);
          this.cd_ptr += 1;
        }
        break;
      case this.cmd.end:
        if(this.memVal() === 0) {
          this.cd_ptr += 1;
        } else {
          var tmp = this.stack.pop();
          if(typeof tmp === 'undefined') {throw this.errorMsg("MISSING_BEGIN_BRACKET")}
          this.cd_ptr = tmp;
        }
        break;
      case this.cmd.write:
        console.log(this.memVal());
        this.bfr_out += this.charOut(this.memVal());
        this.cd_ptr += 1;
        break;
      case this.cmd.read:
        if(this.bfr_ptr >= this.bfr_in.length) {throw this.errorMsg("READ_BUFFER_OVER")}
        this.memVal(this.charIn(this.bfr_in[this.bfr_ptr]),true);
        this.bfr_ptr += 1;
        this.cd_ptr += 1;
        break;
      default:
        this.cd_ptr += 1;
        break;
    }
  }
  memVal(val,overwrite) {
    if(typeof this.memory[this.mem_ptr] === 'undefined') {
      this.memory[this.mem_ptr] = 0;
    }
    if(typeof val === 'undefined') {
      return this.memory[this.mem_ptr];
    } else {
      if(overwrite) {
        this.memory[this.mem_ptr] = val;  
      } else {
        this.memory[this.mem_ptr] += val;        
      }
    }
  }
  genCode(code) {
    if(code === "") {return [""];}
    var regExp = new RegExp("(" +
    "(?:"+this.escapeCmd(this.cmd.next)+")|" +
    "(?:"+this.escapeCmd(this.cmd.prev)+")|" +
    "(?:"+this.escapeCmd(this.cmd.inc)+")|" +
    "(?:"+this.escapeCmd(this.cmd.dec)+")|" +
    "(?:"+this.escapeCmd(this.cmd.begin)+")|" +
    "(?:"+this.escapeCmd(this.cmd.end)+")|" +
    "(?:"+this.escapeCmd(this.cmd.write)+")|" +
    "(?:"+this.escapeCmd(this.cmd.read)+")" +
    ")",'g');
    return code.match(regExp);
  }
  escapeCmd(val) {
    return (val && /[\\^$.*+?()[\]{}|]/g.test(val)) ? val.replace(/[\\^$.*+?()[\]{}|]/g, '\\$&') : val;
  }
  charOut(val) {
    return this.chr[val];
  }
  charIn(val) {
    return this.chr.indexOf(val);
  }
  errorMsg(code) {
    var msg = this.cd_ptr+1 + "文字め：";
    switch (code) {
      case "MEMORY_OVER":
        return msg+"ポインタが"+this.mem_ptr+"をさしてるね。メモリは0から"+(this.memLen-1)+"までしか 見られないよ！";
      case "MISSING_BEGIN_BRACKET":
        return msg+"対応する [ がないよ！"
      case "MISSING_END_BRACKET"  :
        return msg+"対応する ] がないよ！"
      case "READ_BUFFER_OVER" :
        return msg+"入力に もうデータがないよ！"
      default:
        return msg+"よくわからないけど、なにかエラーがおきたよ！"
    }
  }
}

var BF = new BrainFxck("");

export default {
  name: 'App',
  data: function(){
    return {
      result: "",
      code  : "",
      input : "",
      codePos: 0,
      cmd   : {
        next: ">",
        prev: "<",
        inc : "+",
        dec : "-",
        begin: "[",
        end  : "]",
        write: ".",
        read : ","
      }
    }
  },
  methods: {
    exec: function(){
      this.result = BF.execCode(this.code, this.input, this.cmd);
    },
    setCodePos: function(e) {
      this.codePos = e.target.selectionStart;
    },
    addChr: function(v) {
      //var pos = document.getElementById('code').selectionStart;
      this.code = this.code.substr(0,this.codePos) + v + this.code.substr(this.codePos, this.code.length);
      this.codePos += 1;
    }
  }
};

//hello world
//>>>++++[<+++++++++++>-]<<++++++++[<+++++++++>-]<.>+++[<++++++++++>-]<-.+++++++..+++.>>.<++++[<------>-]<.>++++[<++++++>-]<.+++.------.--------.>>-----------.

</script>

<style lang="stylus">
.bl_txt
  display block
  margin 5px
  span
    display block
  input
    width 30px
  textarea
    width 300px
    min-height 200px
  button
    display block
  &Wrap
    display flex  
</style>
