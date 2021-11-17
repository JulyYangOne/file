/*
 * @Description: 校验规则
 * @Author: sunxiaodong
 * @Date: 2019-09-19 20:14:57
 * @LastEditors: zhanglu
 * @LastEditTime: 2020-11-12 11:34:23
 */

// 校验基础类
export class Valid {
  constructor(pattern, message) {
    this.pattern = pattern;
    this.message = message;
  }
  /**
   * @desc 正则校验
   * @param { String } value // 需要校验的 字符串
   * @return {Boolean} 校验结果
   */
  test(value) {
    return this.pattern.test(value);
  }

  /**
   * @desc 配合 element-ui 使用，validator
   * @param { String } 校验不通过时的提示语
   * @return { Function } 返回 element 校验函数
   */
  validator(message = this.message) {
    return (rule, value, cb) => {
      if (value === "" || this.test(value)) {
        cb();
      } else {
        cb(message);
      }
    };
  }

  /**
   * @desc 配合 element-ui 使用，一条 rule
   * @param { Object | {message,trigger} } message: 校验不通过的提示语（有默认值），trigger: 触发校验的事件方法
   * @return { Object } 返回一条校验规则
   */
  rule({ message, trigger } = {}) {
    const rule = { validator: this.validator(message) };

    if (trigger) {
      rule.trigger = trigger;
    }

    return rule;
  }

  /**
   * @desc 配合 element-ui 使用，两条条 rule
   * @param { Object | {requiredMsg,message,trigger} } requiredMsg:校验必填时的提示语（有默认值）， message: 校验不通过的提示语（有默认值），trigger: 触发校验的事件方法
   * @return { Array } 返回两条校验规则
   */
  required({ requiredMsg = "内容不能为空", message, trigger } = {}) {
    const required = {
      required: true,
      message: requiredMsg
    };
    const rule = { validator: this.validator(message) };

    if (trigger) {
      required.trigger = trigger;
      rule.trigger = trigger;
    }

    return [required, rule];
  }
}

// 针对 n + m 的专属生成类
export class NumberValid extends Valid {
  /**
   * @param { Object } [opts = {}] 可配置的选项
   * @param { Number } [opts.integer = 0] 正数位数，小数点前的位数限制，最小为0
   * @param { Number } [opts.decimal = 0] 小数位数，小数后的位数限制，最小为0
   * @param { Boolean } [opts.minus = false] 是否包含负数
   * @return
   */
  constructor({ integer = 0, decimal = 0, minus = false } = {}) {
    integer = Math.max(integer, 0);
    decimal = Math.max(decimal, 0);
    // 限制最大值
    const maxVal = `1${"0".repeat(integer)}`.replace(
      /(\d)(?=(?:\d{3})+$)/g,
      "$1,"
    );

    // 符号位的验证及提示
    let signReg = "";
    let signMsg = "";
    if (minus) signReg = "-?";
    else signMsg = "正";

    // 整数位的验证及提示信息
    let integerReg;
    let integerMsg = `小于${maxVal}的${signMsg}数`;
    if (integer) integerReg = `(([1-9][0-9]{0,${integer - 1}})|0)`;
    else integerReg = "0";

    // 小数位的验证及提示信息
    let decimalReg = "";
    let decimalMsg = "";
    if (decimal) {
      decimalReg = `(\\.[0-9]{1,${decimal}})?`;
      decimalMsg = `，且最多${decimal}位小数`;
    }

    // 生成正则
    const pattern = new RegExp(`^${signReg}${integerReg}${decimalReg}$`);
    super(pattern, integerMsg + decimalMsg);
  }
}

// 针对 n 位 数字 + 字母 的正则
export class NumberOrLetterValid extends Valid {
  /**
   * @param { Object } [opts = {}] 可配置的选项
   * @param { Number } [opts.letterCase = "both"] 是否区分大小写，both：不区分，lower：小写，upper：大写
   * @param { Number } [opts.limit = 1] 限制位数
   */
  constructor({ letterCase = "both", limit = 1 } = {}) {
    limit = Math.max(limit, 1);

    // 符号位的验证及提示
    let letterReg = "";
    let letterMsg = "";
    switch (letterCase) {
      case "low":
        letterReg = "a-z";
        letterMsg = "小写";
        break;
      case "upper":
        letterReg = "A-Z";
        letterMsg = "大写";
        break;
      default:
        letterReg = "A-Za-z";
    }

    // 生成正则
    const pattern = new RegExp(`^[0-9${letterReg}]{${limit}}$`);
    super(pattern, `请输入${limit}位数字/${letterMsg}英文字母`);
  }
}

// 校验是不是由数字组成的字符串
export const stringNumValid = new Valid(/^[0-9]+$/, "输入的内容必须由数字组成");

// 正数 (正整数、正的浮点数)
export const positiveNumValid = new Valid(
  /^0\.\d+$|^[1-9]+(\.\d+)?$/,
  "请输入正数"
);

// 所有合法的数值
export const numberValid = new Valid(
  /^([+-]?\d+)(\.\d+)?$/,
  "请输入正确的数值"
);

// 输入的内容必须由字母组成
export const letterValid = new Valid(/^[a-zA-Z]+$/, "输入的内容必须由字母组成");

// 输入的内容必须由汉字组成
export const zhCNValid = new Valid(
  /^[\u4E00-\u9FA5\uF900-\uFA2D]$/,
  "输入的内容必须由汉字组成"
);

// 输入正确的身份证
export const IDNOValid = new Valid(
  /^(^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$)|(^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[Xx])$)$/,
  "输入正确的身份证"
);

// 邮箱集合（抄送 邮箱）
export const emailsValid = new Valid(
  /^(\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*;)*(\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*;?)?$/,
  "请输入正确的邮箱"
);

// 座机
export const telValid = new Valid(/^[0-9]+-?[0-9]+$/, "请输入正确的固定电话");

// 移动电话
export const mobileValid = new Valid(/^\d{1,20}$/, "请输入正确的手机号码");

// 座机 + 移动电话
export const telAndMobileValid = new Valid(
  /^\d+-?\d+$/,
  "请输入正确的电话号码"
);

// 银行卡号
export const bankNoValid = new Valid(/^\d{12,19}$/, "请输入正确的银行卡号");

// 驾驶证号 目前和身份证一致
export const drivingLicValid = new Valid(
  /^(^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$)|(^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[Xx])$)$/,
  "输入正确的驾驶证号"
);

// 发动机号
export const engineNoValid = new Valid(
  /^[a-zA-Z0-9]+$/,
  "请输入正确的发动机号"
);

// 正整数
export const positiveIntegerValid = new Valid(/^([1-9]\d*)$/, "请输入正整数");

// 12+2位小数
export const positiveDecimal2Valid = new Valid(
  /^([1-9][0-9]{0,11}|0)(\.[0-9]{1,2})?$/,
  "小于1,000,000,000,000的正数，且最多2位小数"
);
// 6位小数
export const decimal6Valid = new Valid(
  /^-?([1-9][0-9]*|0)(\.[0-9]{1,6})?$/,
  "请输入六位小数"
);
// 8+7位小数
export const positiveDecimal7Valid = new Valid(
  /^(([1-9][0-9]{0,7})|0)(\.[0-9]{1,7})?$/,
  "小于100,000,000的正数，且最多7位小数"
);

// 车辆识别代号
export const vehicleIdentifyCodeValid = new NumberOrLetterValid({
  letterCase: "upper",
  limit: 17
});

// 统一社会信用代码
export const SocialCreditCodeValid = new NumberOrLetterValid({
  letterCase: "upper",
  limit: 18
});
