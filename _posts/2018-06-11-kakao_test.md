---
layout: posts
title:  "카카오 코딩테스트"
date:   2018-06-11 22:12:21 +0900
comments: true
categories:
- study
tags:
- 카카오 코딩테스트
redirect_from:
- /kakao/2018/06/11/kakao_test/
- /kakao/kakao_test/
---

### 심심해서 해본 2017 카카오 신입공채 1차 코딩 스트

*제한시간 5시간으로 잡고 카페에서 생각날때마다 풀이를 해봤다.*
##### 결과는 처참..... 4번 문제에서 접근을 잘못해서 시간낭비를 어마어마하게 함 ㅜㅜㅜㅜㅜ

**1번** 비밀 지도(난이도: 하) 정답률 : 81.78%

~~~~~~~javascript
function exam1(len, arr1, arr2){
	var i=0;
	var arr = [];

  function validCheck(num){
    var maxValue = Math.pow(2, len)-1;
    var minValue = 0;
    if(num > maxValue || num < minValue)
      throw "범위 초과";
  }

  function getBinary(num){
    var binary = num.toString(2);
    return binary.padStart(len, 0);
  }

  function lPad(str){
    return str.replace(/0/g, ' ').replace(/1/g, '#');
  }

  for(i;i<len;i++){
    var val1 = arr1[i], val2 = arr2[i];
    validCheck(val1);
    validCheck(val2);
  	var result = getBinary(val1 | val2);
		arr.push(lPad(result));
  }
	return arr;
}

exam1(5, [9, 20, 28, 18, 11], [30, 1, 21, 17, 28]);
exam1(6, [46, 33, 33 ,22, 31, 50], [27 ,56, 19, 14, 14, 10]);
~~~~~~~~~~~~~~~
<br>

**2번** 다트 게임(난이도: 하) 정답률 : 73.47%
```javascript
// 정규식 테스트사이트 참고
function exam2(str){
  var state = 0;
  var regx = /\d+[SDT]([*#]?)+/g;
  var scoreRegx = /\d+/;
  var zoneRegx = /[SDT]/;
  var scores = [];

  function scoreCalc(score, zone, option){
    var currentScore = Math.pow(parseInt(score), {'S':1, 'D':2, 'T':3}[zone]);
    if(option === '*'){
      if(scores[state-1]){
        scores[state-1] *= 2;
      }
      scores[state] = currentScore*2;

    } else if(option === '#'){
      scores[state] = -currentScore;
    } else {
      scores[state] = currentScore;
    }
  }

  function sum(){
    var value = 0;
    scores.forEach(function(obj){
      value += obj;
    });
    return value;
  }

  var val;
  while((val = regx.exec(str)) !== null){
    val = val[0];
    var score = scoreRegx.exec(val)[0];
    var zone = zoneRegx.exec(val)[0];
    var option = val.replace(score+zone, '');
    scoreCalc(score, zone, option);
    state++;
  }
  console.log(str, sum());
}

exam2('1S2D*3T');
exam2('1D2S#10S');
exam2('1D2S0T');
exam2('1S*2T*3S');
exam2('1D#2S*3S');
exam2('1T2D3D#');
exam2('1D2S3T*');
```
<br>

**3번** 캐시(난이도: 하) 정답률 : 45.26% <br>
다 풀어 놓고 저장 안한걸 1주일 후에 알았던 문제
```javascript
function exam3(cacheSize, cities){
  var HIT = 1, MISS = 5;
  var cost = 0;
  var cacheList = [];

  function putCache(name){
    if(cacheList.length >= cacheSize)
      cacheList.splice(0,1);
    cacheList.push(name);
  }

  function getCache(name){
    var point = -1;
    var cache;
    cacheList.forEach(function(data, idx){
      if(name === data){
        point = idx;
      }
    });
    if(point !== -1){
      cost += HIT;
      cacheList.push(cache = cacheList.splice(point,1)[0]);
    }else{
			putCache(name);
      cost += MISS;
    }
    return cache;
  }

  cities.forEach(function(city){
		var _city = city.toLowerCase();
    var data = getCache(_city);
  });
  console.log('소요시간',  cost);
  return cost;

}

exam3(3, ["Jeju", "Pangyo", "Seoul", "NewYork", "LA", "Jeju", "Pangyo", "Seoul", "NewYork", "LA"]);
exam3(3, ["Jeju", "Pangyo", "Seoul", "Jeju", "Pangyo", "Seoul", "Jeju", "Pangyo", "Seoul"]);
exam3(2, ["Jeju", "Pangyo", "Seoul", "NewYork", "LA", "SanFrancisco", "Seoul", "Rome", "Paris", "Jeju", "NewYork", "Rome"]);
exam3(5, 	["Jeju", "Pangyo", "Seoul", "NewYork", "LA", "SanFrancisco", "Seoul", "Rome", "Paris", "Jeju", "NewYork", "Rome"]);
exam3(2, ["Jeju", "Pangyo", "NewYork", "newyork"]);
exam3(0, 	["Jeju", "Pangyo", "Seoul", "NewYork", "LA"]);
```
<br>

**4번** 셔틀버스(난이도: 중) 정답률 : 26.79%

```javascript
/**
입력 형식
셔틀 운행 횟수 n, 셔틀 운행 간격 t, 한 셔틀에 탈 수 있는 최대 크루 수 m, 크루가 대기열에 도착하는 시각을 모은 배열 timetable이 입력으로 주어진다.

0 ＜ n ≦ 10
0 ＜ t ≦ 60
0 ＜ m ≦ 45
timetable은 최소 길이 1이고 최대 길이 2000인 배열로, 하루 동안 크루가 대기열에 도착하는 시각이 HH:MM 형식으로 이루어져 있다.
크루의 도착 시각 HH:MM은 00:01에서 23:59 사이이다.
**/

function exam4(n, t, m, timetable){
	var CON;
	var busStartTime; // 버스운행시간

	timetable.sort(); // 타임테이블 정렬

	function initBustTime(){
		if(busStartTime === undefined){
			busStartTime = new Date(0);
			busStartTime.setHours(9);
			busStartTime.setMinutes(0);
		} else{
			busStartTime.setMinutes(busStartTime.getMinutes()+t);
		}
	}

	function initTime(str){
		var time = new Date(0);
		var split = str.split(':');
		time.setHours(split[0]);
		time.setMinutes(split[1]);
		return time;
	}

	var lastTime; // 마지막 탑승시각
	var isFull; // 만석여부
	var driveCnt = 0; // 운행횟수
	while(driveCnt++ < n){
		var cnt = 0; // 탑승인원
		isFull = false;
		initBustTime(); // 버스운행시간 초기화
		var i=0; len = timetable.length, clone = Array.from(timetable);
		for(;i<len;i++){
			var waitter = initTime(clone[i]);
			if(waitter <= busStartTime && cnt < m){
				if(++cnt === m){
					isFull = true;
					lastTime = waitter;
					break;
				}
			}
		}
		timetable.splice(0, cnt);
	}
	if(isFull){
		lastTime.setMinutes(lastTime.getMinutes()-1);
		CON = lastTime
	}else{
		CON = busStartTime;
	}

	CON = CON.getHours().toString().padStart(2,0) + ':' + CON.getMinutes().toString().padStart(2,0);
	console.log(CON);
	return CON;
}

exam4(1, 1, 5, ["08:00", "08:01", "08:02", "08:03"]);
exam4(2, 10, 2, ["09:10", "09:09", "08:00"]);
exam4(2, 1, 2, ["09:00", "09:00", "09:00", "09:00"]);
exam4(1, 1, 5, ["00:01", "00:01", "00:01", "00:01", "00:01"]);
exam4(1, 1, 1, ["23:59"]);
exam4(10, 60, 45, ["23:59","23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59", "23:59"]);

```
<br>

**5번** 뉴스 클러스터링(난이도: 중) 정답률 : 41.84%
```javascript
/**
 * 자카드 유사도 = 교집합/합집합
 * 두집합 모드 공집합일경우 유사도를 따로 1로 정의
 * 입력값은 영문자로 된 글자 쌍만 유효
 * 기타 공백이나 숫자, 특수문자가 들어가 있는경우  그 글자 쌍을 버린다.
 */
function exam5(str1, str2){
	var setA;
	var setB;
	var union;
	var intersection = 0;
	var regx = /[a-zA-Z][a-zA-Z]/;


	function createSet(s){
		var arr = [];
		var len = s.length;
		for(var i = 0; i < len-1; i++){
			var str = s.substring(i, i+2);
			if(regx.test(str))
				arr.push(str.toLowerCase());
		}
		return arr;
	}

	setA = createSet(str1);
	setB = createSet(str2);

	var unionSet = [];
	var overGroup = {};
	setA.forEach(function(ele){
		unionSet.push(ele);
		if(overGroup[ele] === undefined)
			overGroup[ele] = 1;
		else
			overGroup[ele]++;
	});

	var interSectionSet = [];
	setB.forEach(function(ele){
		var cnt = 0;
		if(overGroup[ele] === undefined){
			unionSet.push(ele);
		}else{
			if(overGroup[ele] > 0){
				overGroup[ele]--;
				interSectionSet.push(ele);
			}else{
				unionSet.push(overGroup[ele]);
			}
		}
	});

	union = unionSet.length;
	intersection = interSectionSet.length

	var result
	if(union === 0)
		result = 1;
	else
 		result = Math.floor((intersection / union) * 65536);
	console.log('union is ', union, 'intersection is ', intersection, 'result is ', result);

	return result;
}

exam5('FRANCE', 'french');
exam5('handshake', 'shake hands');
exam5('aa1+aa2', 'AAAA12');
exam5('E=M*C^2', 'e=m*c^2');
```
<br>



포스팅 하고보니..... 들여쓰기 무엇..?
