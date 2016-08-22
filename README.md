# help.js


'var storage = (function() {
  var isSupport = function(storageType) {
    var testKey = 'test',
      storage = window[storageType];
    try {
      storage.setItem(testKey, '1');
      storage.removeItem(testKey);
      return storageType in window && window[storageType];
    } catch (e) {
      return false;
    }
  }
  var storages = ['localStorage', 'sessionStorage'];
  for (var i = 0, len = storages.length; i < len; i++) {
    if (isSupport(storages[i])) return window[storages[i]];
  }
  return null;
})();

const helper = {
  storage: storage,
  /* 判断 obj 是否为 null
     Usage:
       var obj = null;
       helper.isNull(obj); // 输出 true
  */
  isNull: function(obj) {
    return obj === null || obj !== obj;
  },

  /* 判断 obj 是否为 function
     Usage:
       var obj = 'abc';
       helper.isFunction(obj); // 输出 false
  */
  isFunction: isType('Function'),

  /* 判断 obj 是否为 object
     Usage:
       var obj = {};
       helper.isObject(obj); // 输出 true
  */
  isObject: isType('Object'),

  /* 判断 obj 是否为 array
     Usage:
       var obj = ['abc'];
       helper.isArray(obj); // 输出 true
  */
  isArray: window.isArray || isType('Array'),

  /* 判断 obj 是否为 string
     Usage:
       var obj = 'abc';
       helper.isString(obj); // 输出 true
  */
  isString: isType('String'),

  /* 判断 obj 是否为 undefined
     Usage:
       var obj = 'abc';
       helper.isUndefined(obj); // 输出 false
  */
  isUndefined: isType('Undefined'),

  // 获取 Cookie
  getCookie: getCookie,
  // 设置 Cookie
  setCookie: setCookie,
  // 移除 Cookie
  removeCookie: removeCookie,

  /* 判断请求是否成功
      @param res `Object` 后台返回的response对象
     Usage:
      helper.isSuccess(res)
  */
  isSuccess: function(res) {
    return Number(res.status.code) == 0;
  },

  /* 获取浏览器存储里key为item的值
      @param item 要获取的数据的索引值
     Usage:
       helper.get('token');
  */
  get: function(item) {
    var value;
    if (this.storage) {
      value = this.storage.getItem(item);
    } else {
      value = getCookie(item);
    }
    return (value ? this.decrypt(value) : '');
  },

  /* 将数据存储在浏览器存储里
      @param obj 要存储的数据对象
     Usage:
       j.set({ token: 'as23q1sdf212swsxx', uname: 'jacket' });
  */
  set: function(obj) {
    for (var key in obj) {
      if (obj.hasOwnProperty(key)) {
        this.storage ? this.storage.setItem(key, this.encrypt(String(obj[key]))) :
          setCookie(key, this.encrypt(String(obj[key])), 1);
      }
    }
    return this;
  },

  /* 删除浏览器存储的数据
      @param itemArr 要删除的数据的key组成的数组
     Usage:
      j.remove(['token', 'uname']); // 将删除浏览器存储中，索引为token和uname的数据
  */
  remove: function(itemArr) {
    for (var i = 0, len = itemArr.length; i < len; i++) {
      if (this.storage) {
        this.storage.removeItem(itemArr[i]);
      } else {
        removeCookie(itemArr[i]);
      }
    }
    return this;
  },

  /* 获取search的某个值
     Usage:
       页面url为http://a.com?i=1&j=2&k=3, j.getSearchItem('k') 将输出'3'
  */
  getSearchItem: function(key) {
    var o = this.getSearch();
    return o[key] ? decodeURIComponent(o[key]) : null;
  },

  // 加密
  encrypt: function(value) {
    var encryptValue = new String,
      mask, maskValue;
    for (var i = 0, len = value.length; i < len; i++) {
      mask = Math.round(Math.random() * 111) + 77;
      maskValue = value.charCodeAt(i) + mask;
      mask += i;
      encryptValue += String.fromCharCode(maskValue, mask);
    }
    return encryptValue;
  },

  // 解密
  decrypt: function(value) {
    var decryptValue = new String,
      k = 0,
      v, m;
    var decrypt = function(v, m, i) {
      v = v.charCodeAt(0);
      m = m.charCodeAt(0);
      m -= i;
      v -= m;
      return String.fromCharCode(v);
    }
    for (var i = 0, len = value.length; i < len; i++) {
      if (!(i % 2)) {
        v = value[i];
      } else {
        m = value[i];
        decryptValue += decrypt(v, m, k);
        k++;
      }
    }
    return decryptValue;
  }
}

function isType(type) {
  return function(obj) {
    return {}.toString.call(obj) == '[object ' + type + ']';
  }
};

/* 获取 Cookie 值
 */
function getCookie(name) {
  var c = document.cookie;
  if (c.length > 0) {
    var s = c.indexOf(name + '=');
    if (s != -1) {
      s = s + name.length + 1;
      var e = c.indexOf(';', s);
      return unescape(c.substring(s, e));
    }
  }
  return '';
};

/* 保存 Cookie 值
 */
function setCookie(name, value, expiredays) {
  var exdate = new Date();
  if (value != null && value != '' && value != 'null') {
    exdate.setDate(exdate.getDate() + expiredays);
  } else {
    exdate.setDate(exdate.getDate() - 1);
  }
  document.cookie = name + '=' + escape(value) + ((expiredays == null) ? '' : ';expires=' + exdate.toGMTString());
};

/* 删除 Cookie
 */
function removeCookie(name) {
  setCookie(name, '', -1);
};

export default helper;
'
