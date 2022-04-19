---
layout: post
title: 주니어 개발자의 코드개선 프로젝트 - 1
---

안녕하세요. **티켓예약개발팀**의 막내, 주니어 개발자 배성민 입니다! 🖐

입사한지 반년 정도 되었고 이제 막 업무에 적응해가고 있습니다.

이번 이벤트 페이지 제작을 하며 느낀 경험을 공유하려 합니다.

---

MVC 패턴의 페이지였고 특정 기간별로 다른 영역이 노출되는 기능이 포함된 프로젝트를 진행했습니다.

> **"매년 반복되는 작업이니 기존 소스 참고하시면 됩니다."**
> 

기존 작업물이 있는 상황이라 문구나 URL 같은 디테일 수준만 수정할 계획으로 개발에 착수했는데

세상에나. “**각 페이지**"마다 아래와 같이 날짜를 세팅하고 있었습니다.

![상상도못한정체.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/93b138fa-8ace-4f3a-b593-3f1c18726035/상상도못한정체.jpg)

---

```csharp
// Razor
@{

DateTime toDay = DateTime.Now;

DateTime limitDate = new DateTime(2019, 4, 5, 23, 59, 59); // 투표 마감

DateTime releaseDate = new DateTime(2019, 4, 16, 15, 59, 59); // 수상자 발표

DateTime releaseMovieDate = new DateTime(2019, 4, 25, 10, 0, 0); // 수상자 인터뷰 공개

var tYear = toDay.Year;

var tMonth = toDay.Month - 1;

var tDay = toDay.Day;

var thour = toDay.Hour;

var tMinute = toDay.Minute;

var tSecond = toDay.Second;

var bEndYN = (toDay >= limitDate) ? true : false;

var bReleaseYN = (toDay >= releaseDate) ? true : false;

var bReleaseMovieYN = (toDay >= releaseMovieDate) ? true : false;

.

.

.(중략)

}
```

거의 10개쯤 되는 페이지에서 같은 코드를 사용하고 있었고

**심지어** script 쪽에도 따로 날짜를 세팅하고 있었습니다.

```csharp
// script
function setDday()

{

today = new Date();

d_day = new Date(2019, 03, 5, 23, 59, 59);

days = (d_day - today) / 1000 / 60 / 60 / 24;

.

.

.(중략)

}
```

---

전체적으로 같은 기간에 따라 표시되는 영역이 제어되므로

기간이 변경되는 경우, 각 페이지와 script를 일일이 찾아 수정해야 하고

그중 하나라도 빼먹는 실수를 저지른다면 치명적인 휴먼에러가 발생할 것 같아

기간 세팅부터 **공용으로 사용**해야겠다고 생각했습니다.

---

MVC 패턴이고 Front 단에서 Razor 문법을 이용하여 Server Side 변수를 사용할 수 있었기 때문에

Controller 단에서 작업 후 **ViewBag으로 넘겨주기** 혹은 공통으로 사용하는 모델에서 작업을 진행 후 해당 값을 가져다 쓰기로 했습니다.

첫 번째로 변수명에 대한 수정을 진행했습니다.

bEndYN, bReleaseYN, bReleaseMovieYN 와 같은 변수명은 직관적이지 않고, 전체적으로 눈에 들어오지 않았기 때문에 Enum Type의 Class를 사용하여 기간 별 노출되는 페이지 혹은 동작들을 한눈에 알아볼 수 있도록 수정했습니다.

각 페이지에서 공통적으로 사용하는 Model에 아래와 같이 Enum Class를 정의하였고

해당 변수(Phase)를 사용하는 방식으로 진행했습니다.

```csharp
public enum Phase

{

	투표중_01 = 1,

	투표마감_02 = 2,

	수상자발표_03 = 3

}
```

또한 각 Phase 별 작업을 분리해야 했기 때문에

현재 Phase 담는 변수를 만든 후 한 번의 함수 호출로 현재 Phase 값을 담을 수 있도록 했습니다.

```csharp
[Front]

Phase CurrentPhase = Model.GetPhase();

[Back]

public Phase GetPhase()

{

	Phase resultPhase = 0;

	DateTime Today = DateTime.Now;
	DateTime phase_1_Date = new DateTime(2022, 4, 8, 23, 59, 59); // 투표마감(Close Vote)
	DateTime phase_2_Date = new DateTime(2022, 4, 18, 15, 59, 59); // 수상자 발표(Announce)

	// 투표 중 단계
	if (Today <= phase_1_Date)
	{
		resultPhase = Phase.투표중_01;
	}

	// 투표 마감 단계
	if (Today > phase_1_Date && Today <= phase_2_Date)
	{
		resultPhase = Phase.투표마감_02;
	}

	// 수상자 발표
	if (Today > phase_2_Date)
	{
		resultPhase = Phase.수상자발표_03;
	}

	return resultPhase;
}
```

---

주석 없이는 한눈에 알 수 없었던 코드에서 **한눈에 알아볼 수 있는 코드**로 변경이 완료됐습니다!

이제 이벤트 기간이 변경되어도 한 군데만 수정하게 되면 전체 페이지에 적용이 됩니다!

## ASIS

```csharp
// ASIS
@if (bEndYN && !bReleaseYN) // 투표 마감
{
	//someting..
}

else if (bReleaseYN) // 수상자 발표

{
	//someting..
}

else // 투표 진행
{
	//someting..
}
```

## TOBE

```csharp
// TOBE
@if (CurrentPhase == Phase.투표중_01)
{
	//someting..
}
@if (CurrentPhase == Phase.투표마감_02)
{
	//someting..
}
@if (CurrentPhase == Phase.수상자발표_03)
{
	//someting..
}
```

반복되어 진행되는 이벤트인 만큼 유지 보수 관점에서 간단하게 기간 변경이 가능하며 휴먼에러를 줄일 수 있는 작업이 완료되었습니다.

반복적인 cs 업무를 진행하다 보면 특정 케이스 별로 업체 코드, 상품코드 등을 하드코딩하는 경우가 많았습니다.

별것 없는 작업이었지만 이런 식으로 코드를 개선해나가면 시간이 지난 후엔 유지 보수에 들어가는 리소스가 줄어들 것이고, 남는 공수를 활용하여 다른 작업 시 퀄리티가 더욱 높아질 수 있을 거라 생각합니다.

![l1ilfO33.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/09c46b77-cec8-4260-9a30-d5eafc829562/l1ilfO33.jpg)
