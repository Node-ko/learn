---
title: Node.js 애플리케이션 프로파일링
layout: learn
---

# Node.js 애플리케이션 프로파일링
> ❗️ *번역 날짜: 2024년 12월 03일* <br>
> 공식 문서 원문은 아래를 참고하세요.<br>
> [Profiling Node.js Applications](https://nodejs.org/en/learn/getting-started/profiling)

Node.js 애플리케이션의 프로파일링은 애플리케이션 실행 중에 CPU, 메모리 및 기타 런타임 지표를 분석하여 성능을 측정하는 작업을 의미합니다. 이를 통해 애플리케이션의 효율성, 응답성, 확장성에 영향을 미칠 수 있는 병목 현상, 높은 CPU 사용률, 메모리 누수 또는 느린 함수 호출을 식별할 수 있습니다.

Node.js 애플리케이션을 프로파일링하기 위한 서드파티 도구가 많이 있지만, 대부분의 경우 가장 간단한 방법은 Node.js의 내장 프로파일러를 사용하는 것입니다.
내장 프로파일러는 [V8 내부의 프로파일러][]를 사용하며, 프로그램 실행 중 일정한 간격으로 스택을 샘플링합니다.
이 샘플링 결과와 JIT 컴파일과 같은 중요한 최적화 이벤트를 **틱(ticks)** 의 형태로 기록합니다:

```
code-creation,LazyCompile,0,0x2d5000a337a0,396,"bp native array.js:1153:16",0x289f644df68,~
code-creation,LazyCompile,0,0x2d5000a33940,716,"hasOwnProperty native v8natives.js:198:30",0x289f64438d0,~
code-creation,LazyCompile,0,0x2d5000a33c20,284,"ToName native runtime.js:549:16",0x289f643bb28,~
code-creation,Stub,2,0x2d5000a33d40,182,"DoubleToIStub"
code-creation,Stub,2,0x2d5000a33e00,507,"NumberToStringStub"
```

과거에는 **틱(ticks)** 을 해석하기 위해 V8 소스 코드를 별도로 사용해야 했습니다.
다행히도, Node.js 4.4.0 이후부터는 이러한 정보를 활용하기 위해 V8을 소스 코드에서 빌드하지 않아도 되는 도구들이 도입되었습니다.
이제 내장 프로파일러가 애플리케이션 성능에 대한 통찰을 제공하는 데 어떻게 도움이 되는지 알아보겠습니다.

틱 프로파일러의 사용법을 설명하기 위해 간단한 Express 애플리케이션을 사용해 보겠습니다.
이 애플리케이션은 두 개의 핸들러를 가지며, 그 중 하나는 시스템에 새로운 사용자를 추가하는 역할을 합니다:

```js
app.get('/newUser', (req, res) => {
  let username = req.query.username || '';
  const password = req.query.password || '';

  username = username.replace(/[!@#$%^&*]/g, '');

  if (!username || !password || users[username]) {
    return res.sendStatus(400);
  }

  const salt = crypto.randomBytes(128).toString('base64');
  const hash = crypto.pbkdf2Sync(password, salt, 10000, 512, 'sha512');

  users[username] = { salt, hash };

  res.sendStatus(200);
});
```

또 다른 하나는 사용자 인증 시도를 검증하는 역할을 합니다:

```js
app.get('/auth', (req, res) => {
  let username = req.query.username || '';
  const password = req.query.password || '';

  username = username.replace(/[!@#$%^&*]/g, '');

  if (!username || !password || !users[username]) {
    return res.sendStatus(400);
  }

  const { salt, hash } = users[username];
  const encryptHash = crypto.pbkdf2Sync(password, salt, 10000, 512, 'sha512');

  if (crypto.timingSafeEqual(hash, encryptHash)) {
    res.sendStatus(200);
  } else {
    res.sendStatus(401);
  }
});
```

_이 핸들러들은 Node.js 애플리케이션에서 사용자를 인증하기 위한 권장 핸들러가 아님을 유의하세요.
이 핸들러들은 단순히 설명을 위해 사용되는 예제일 뿐입니다.
일반적으로 직접 암호화 인증 메커니즘을 설계하려고 하지 않는 것이 좋습니다. 대신 기존의 검증된 인증 솔루션을 사용하는 것이 훨씬 안전합니다._

이제 애플리케이션을 배포했다고 가정해 보겠습니다. 사용자들이 요청 시 높은 지연(latency)이 발생한다고 불만을 제기하고 있습니다.
내장 프로파일러를 사용하면 애플리케이션을 쉽게 실행하여 문제를 분석할 수 있습니다:

```
NODE_ENV=production node --prof app.js
```

그리고 `ab` (ApacheBench)를 사용하여 서버에 부하를 가해봅니다:

```
curl -X GET "http://localhost:8080/newUser?username=matt&password=password"
ab -k -c 20 -n 250 "http://localhost:8080/auth?username=matt&password=password"
```

그리고 다음과 같은 ab 출력 결과를 얻습니다:

```
Concurrency Level:      20
Time taken for tests:   46.932 seconds
Complete requests:      250
Failed requests:        0
Keep-Alive requests:    250
Total transferred:      50250 bytes
HTML transferred:       500 bytes
Requests per second:    5.33 [#/sec] (mean)
Time per request:       3754.556 [ms] (mean)
Time per request:       187.728 [ms] (mean, across all concurrent requests)
Transfer rate:          1.05 [Kbytes/sec] received

...

Percentage of the requests served within a certain time (ms)
  50%   3755
  66%   3804
  75%   3818
  80%   3825
  90%   3845
  95%   3858
  98%   3874
  99%   3875
 100%   4225 (longest request)
```

이 출력 결과에서 우리는 초당 약 5개의 요청만 처리할 수 있으며, 요청 하나를 처리하는 데 평균적으로 약 4초가 소요된다는 것을 알 수 있습니다.
실제 사례에서는 사용자 요청을 처리하기 위해 많은 함수에서 다양한 작업을 수행하겠지만, 심지어 이 간단한 예제에서도 시간을 소모할 수 있는 부분은 많습니다.
예를 들어, 정규식을 컴파일하거나, 랜덤 솔트를 생성하거나, 사용자 비밀번호로부터 고유한 해시를 생성하거나, Express 프레임워크 내부에서 발생하는 작업들이 그 예입니다.

`--prof` 옵션을 사용하여 애플리케이션을 실행했기 때문에, 애플리케이션을 로컬에서 실행한 디렉터리에 **틱 파일(tick file)** 이 생성되었습니다. 이 파일의 이름은 `isolate-0xnnnnnnnnnnnn-v8.log` 형식을 따르며, 여기서 `n`은 숫자를 나타냅니다.

이 파일을 해석하려면 Node.js 바이너리에 포함된 **틱 프로세서(tick processor)** 를 사용해야 합니다. 틱 프로세서를 실행하려면 `--prof-process` 플래그를 사용합니다:

```
node --prof-process isolate-0xnnnnnnnnnnnn-v8.log > processed.txt
```

processed.txt 파일을 당신이 선호하는 텍스트 편집기로 열면 다양한 유형의 정보를 확인할 수 있습니다.
이 파일은 여러 섹션으로 나뉘어 있으며, 각 섹션은 언어별로 다시 구분됩니다.
먼저, 요약 섹션(summary section)을 살펴보면 다음과 같은 내용을 확인할 수 있습니다:

```
 [Summary]:
   ticks  total  nonlib   name
     79    0.2%    0.2%  JavaScript
  36703   97.2%   99.2%  C++
      7    0.0%    0.0%  GC
    767    2.0%          Shared libraries
    215    0.6%          Unaccounted
```

이는 수집된 샘플의 97%가 C++ 코드에서 발생했음을 의미하며, 처리된 출력의 다른 섹션을 볼 때, JavaScript보다 C++에서 수행되는 작업에 더 많은 주의를 기울여야 한다는 것을 알려줍니다. 이를 염두에 두고, 다음으로 \[C++\] 섹션을 찾아  C++ 함수 중에서 어떤 함수가 가장 많은 CPU 시간을 소비하고 있는지 확인합니다:

```
 [C++]:
   ticks  total  nonlib   name
  19557   51.8%   52.9%  node::crypto::PBKDF2(v8::FunctionCallbackInfo<v8::Value> const&)
   4510   11.9%   12.2%  _sha1_block_data_order
   3165    8.4%    8.6%  _malloc_zone_malloc
```

이 출력에서 상위 3개의 항목이 프로그램의 CPU 사용 시간 중 72.1%를 차지한다는 것을 알 수 있습니다. 특히, 사용자 비밀번호로부터 해시를 생성하는 함수인 PBKDF2가 전체 CPU 시간의 최소 51.8%를 소비하고 있다는 것을 바로 확인할 수 있습니다. 그러나 나머지 두 항목이 애플리케이션에서 어떤 방식으로 연관되어 있는지는 즉시 명확하지 않을 수 있습니다(또는 예제 설명을 위해 그렇지 않다고 가정하겠습니다). 
이 함수들 간의 관계를 더 잘 이해하기 위해, 각 함수의 주요 호출자를 보여주는 \[Bottom up (heavy) profile\] 섹션을 살펴보겠습니다.
이 섹션을 검사한 결과, 다음과 같은 내용을 확인합니다:

```
   ticks parent  name
  19557   51.8%  node::crypto::PBKDF2(v8::FunctionCallbackInfo<v8::Value> const&)
  19557  100.0%    v8::internal::Builtins::~Builtins()
  19557  100.0%      LazyCompile: ~pbkdf2 crypto.js:557:16

   4510   11.9%  _sha1_block_data_order
   4510  100.0%    LazyCompile: *pbkdf2 crypto.js:557:16
   4510  100.0%      LazyCompile: *exports.pbkdf2Sync crypto.js:552:30

   3165    8.4%  _malloc_zone_malloc
   3161   99.9%    LazyCompile: *pbkdf2 crypto.js:557:16
   3161  100.0%      LazyCompile: *exports.pbkdf2Sync crypto.js:552:30
```

이 섹션을 해석하는 것은 위의 원시 틱(raw tick) 수를 해석하는 것보다 조금 더 많은 작업이 필요합니다.
위의 각 “call stacks”에서, 부모 열(parent column)에 표시된 퍼센트 값은 현재 행의 함수가 바로 위 행의 함수에 의해 호출된 샘플의 비율을 나타냅니다. 예를 들어, 중간 "call
stack" 에서 \_sha1_block_data_order의 경우, `_sha1_block_data_order`가 샘플의 11.9%에서 발생했음을 보여줍니다. 이는 위에서 확인한 원시 틱 수와 일치합니다. 그러나 여기에서는 _sha1_block_data_order가 항상 Node.js crypto 모듈 내부의 pbkdf2 함수에 의해 호출되었다는 것도 알 수 있습니다. 마찬가지로, `_malloc_zone_malloc` 또한 거의 예외 없이 동일한 pbkdf2 함수에 의해 호출되었음을 확인할 수 있습니다. 따라서, 이 뷰에서 얻은 정보를 통해 사용자 비밀번호로부터 해시를 계산하는 작업이 단순히 위에서 언급한 51.8%만 차지하는 것이 아니라, 샘플링된 상위 3개 함수의 모든 CPU 시간도 포함하고 있음을 알 수 있습니다. 이는 `_sha1_block_data_order`와
`_malloc_zone_malloc` 호출이 모두 pbkdf2 함수를 대신하여 수행되었기 때문입니다.

이 시점에서, 비밀번호 기반 해시 생성이 최적화의 대상이 되어야 한다는 것이 매우 명확합니다.
다행히도, 당신은 [비동기 프로그래밍의 이점][]을 충분히 이해하고 있으며, 사용자의 비밀번호로 해시를 생성하는 작업이 동기 방식으로 수행되고 있어 이벤트 루프를 차단하고 있다는 사실을 깨닫습니다.
이로 인해 해시를 계산하는 동안 다른 들어오는 요청을 처리할 수 없게 됩니다.

이 문제를 해결하기 위해, 위 핸들러를 약간 수정하여 pbkdf2 함수의 비동기 버전을 사용하도록 변경합니다:

```js
app.get('/auth', (req, res) => {
  let username = req.query.username || '';
  const password = req.query.password || '';

  username = username.replace(/[!@#$%^&*]/g, '');

  if (!username || !password || !users[username]) {
    return res.sendStatus(400);
  }

  crypto.pbkdf2(
    password,
    users[username].salt,
    10000,
    512,
    'sha512',
    (err, hash) => {
      if (users[username].hash.toString() === hash.toString()) {
        res.sendStatus(200);
      } else {
        res.sendStatus(401);
      }
    }
  );
});
```

애플리케이션의 비동기 버전으로 변경한 후, 위에서 사용한 ab 벤치마크를 다시 실행하면 다음과 같은 결과가 나타납니다:

```
Concurrency Level:      20
Time taken for tests:   12.846 seconds
Complete requests:      250
Failed requests:        0
Keep-Alive requests:    250
Total transferred:      50250 bytes
HTML transferred:       500 bytes
Requests per second:    19.46 [#/sec] (mean)
Time per request:       1027.689 [ms] (mean)
Time per request:       51.384 [ms] (mean, across all concurrent requests)
Transfer rate:          3.82 [Kbytes/sec] received

...

Percentage of the requests served within a certain time (ms)
  50%   1018
  66%   1035
  75%   1041
  80%   1043
  90%   1049
  95%   1063
  98%   1070
  99%   1071
 100%   1079 (longest request)
```

야호! 이제 애플리케이션이 초당 약 20개의 요청을 처리하고 있으며, 이는 동기 해시 생성 방식일 때보다 약 4배 증가한 수치입니다.
게다가 평균 지연 시간(latency)도 이전의 4초에서 약 1초로 크게 줄어들었습니다.

이 (다소 인위적인) 예제를 통해 성능 조사를 진행하면서 V8 틱 프로세서가 Node.js 애플리케이션의 성능을 더 잘 이해하는 데 얼마나 유용한 도구인지를 확인하셨기를 바랍니다.

[플레임 그래프(flame graph)를 생성하는 방법][진단용 플레임 그래프]도 유용할 수 있습니다.

[V8 내부의 프로파일러]: https://v8.dev/docs/profile
[비동기 프로그래밍의 이점]: https://nodesource.com/blog/why-asynchronous
[진단용 플레임 그래프]: /learn/diagnostics/flame-graphs