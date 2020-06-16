---
title: "[개발기록] RabbitMQ, EC2 메모리 부족 문제"
subtitle: "EC2 - No space left on device"
categories:
  - 개발기록
tags:
  - 개발기록
  - aws
  - EC2
  - RabbitMQ
classes: wide
---

## 문제 발생

EC2에 RabbitMQ를 설치하여 사용하던 도중, 서버에서 메세지를 정상적으로 퍼블리시 했으나 큐에서 메세지를 받지 못하는 상황이 발생했다.(분명 메세지를 보냈지만 큐에는 쌓이지 않는 이상한 상황!)  
처음에는 RabbitMQ와 node 서버 간의 연결 문제로 생각했으나, `connection` 과 `publish`에 아무 문제도 없었다.

RabbitMQ의 log를 보니 아래와 같은 `status=73`이라는 상태 값이 찍혀있는 것을 볼 수 있었고,

```text
Process: 2795 ExecStart=/usr/lib/rabbitmq/bin/rabbitmq-server (code=exited, status=73)
```

해당 코드는 RabbitMQ에서 메모리가 부족할 경우 발생하는 에러라고 한다.

## 삽질🤺

`status=73 -> rabbitmq 메모리 부족` 이라는 문제에 빠져서 **RabbitMQ 자체의 메모리를 정리** 하려고 애를 썼다.
이러한 에러가 발생할 경우 `/mnesia` 를 삭제 후 다시 실행해보라는 포스트를 보고 시도했으나,

```text
sudo rm -rf /var/lib/rabbitmq/mnesia
```

번번히 실패!(에러 로그 스크린샷을 찍어놨으면 좋았을텐데 너무 급하게 해결하다 보니 하나도 찍지 못했다😫)

`mnesia` 디렉토리를 이용한 이런 저런 시도를 하던 중 해당 디렉토리를 수동으로 만들기 위해

```text
sudo mkdir /mnesia
```

명령어를 치는 순간 진짜 문제가 무엇인지 알게 되었다. 아래와 같은 에러가 발생한 것이다.

```text
no space left on device...
```

즉, 해당 RabbitMQ가 실행되고 있는 **EC2 자체의 용량이 꽉 차 있었던 것**...
(위에서 메모리 문제라는 걸 알았을때, 바로 이 단계로 왔으면 좋았을 텐데😭)

## 문제 해결

```text
df -h
```

[linux 하드 디스크 사용량 명령어](https://blog.naver.com/maron0614/120159604933)

확인해 본 결과 실제 디스크 용량이 100% 꽉 차 더 이상 무언가 write 작업이 수행 될 수 없는 상태였다.(이래서 디렉토리 생성이 불가했던 것)

어떤 파일이 용량을 많이 잡아먹었나 찾아보니, `/usr/src` 하위에 EC2 커널 들이 꽉꽉 들어차 있었다.🔥

> Ubuntu를 사용하면서 업데이트, 업그레이드를 진행하다 보면 이처럼 오래된 커널들이 쌓이게 되는데, 메모리 절약을 위해서는 오래된 커널들을 삭제하는 등 관리가 필요하다고 한다!

```text
apt-get autoremove
```

> `apt-get autoremove` 명령어를 사용하면 필요없는 모듈(의존성은 사라졌지만 유지되고 있는 모듈)을 자동으로 삭제해준다.

사용되지 않는 커널 삭제를 위해 명령어를 사용했으나, 역시나 **용량 부족 에러**가 뜨면서 실패...

최후의 방법으로 **EC2 이미지의 용량**을 늘려 새로운 인스턴스를 생성하는 방식을 시도했으나 이상하게 용량이 늘어나지 않았다.

알고보니 EC2 볼륨 크기를 조정 한 후에는 **디스크 용량을 기존 파티션**에 할당해야하는 작업을 해줘야 늘어난 용량이 현재 파티션에 적용된다고 한다.

- [볼륨 크기 조정 후 Linux 파일 시스템 확장](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/recognize-expanded-volume-linux.html)
- [https://djangoworld.tistory.com/17](https://djangoworld.tistory.com/17)

두가지 내용을 참조하여 진행했으나 또 다시 'no space left on device..'!!! (ㅂㄷㅂㄷ)
심지어 재부팅 해도 적용되지 않았다...

```text
apt-get clean
```

> `/var/cache/apt/archives` 경로에 저장된 패키지, 아카이브 파일 삭제 명령어

결국, 위 명령어로 다운 받은 파일을 정리하여 약간의 디스크 공간을 확보한 후에야 용량을 할당을 할 수 있었고, `/mnesia` 디렉토리를 생성한 후 RabbitMQ를 재부팅하니 정상적으로 메시지가 들어오는 것을 확인할 수 있었다.

> [https://docs.openstack.org/openstack-ansible/pike/admin/maintenance-tasks/rabbitmq-maintain.html#rabbitmq-and-mnesia](https://docs.openstack.org/openstack-ansible/pike/admin/maintenance-tasks/rabbitmq-maintain.html#rabbitmq-and-mnesia)  
> mnesia는 rabbitmq 에서 정보 저장을 위해 사용하는 DB라는 것 같다...

### 우분투에서는 왜 오래된 커널을 자동으로 삭제해 주지 않을까?

'오래된' 의 정의가 명확하지 않아서 인 것 같다. '오래됐다 -> 사용하지 않는다' 가 아니므로 시스템에서 자동으로 삭제하지 않는 것이다. 사용되는 커널이 삭제 됐을때 시스템이 제대로 작동하지 않을 것이기 때문에!

- [오래된 커널 삭제](https://extrememanual.net/9326)
