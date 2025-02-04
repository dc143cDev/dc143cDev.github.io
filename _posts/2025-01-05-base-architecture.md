---
layout: post
title: "Flutter 베이스 아키텍처로 일 쉽게 만들기"
date: 2025-01-05
tags: [Flutter, Architecture]
read_time: 30
subtitle: "Getx와 Floor를 활용하여 실전 업무에 적용할 수 있는 아키텍처를 만들어봅니다."
---

## 개요
개발자는 신기한 직업입니다. 일을 줄이려고 일을 만들어내는 직업이기 때문입니다.

심지어 어떨 때는 일을 줄이려고 기반을 작업하는 일에 본격적인 개발 과정보다 더 많은 시간을 소모하기도 합니다.

입사 후 본격적인 실무 작업에 투입된 후 가장 신기했던 경험은, 페이지 단위의 모듈이나 UI 컴포넌트를 작업하는 시간이 아키텍처를 작업하는 시간보다 적거나 비슷하다는 점이었습니다.

디자인에도 관심이 많고, 보이지 않는 부분보다 보이는 부분을 만드는 것에 더 흥미를 느끼는 저로서는 당시의 작업 방향이 의아했지만, 현재로서는 이해가 됩니다.

체계적으로 설계된 아키텍처는 컴포넌트나 페이지 구현같은 UI 작업을 더 쉽게 만들어줄 뿐만 아니라, 비즈니스 로직과 통신이 필요한 작업도 더 안정적이고 정확하게 만들어줍니다.

한마디로 조금만 고생하면 일이 확연히 줄어든다는 것입니다. 아키텍처 하나만 잘 만들어 두면 집에 더 빨리 들어갈 수 있습니다. 이것보다 중요한게 있을까요?

이번 포스트에서는 GetX로 상태 관리를, Floor로 데이터베이스와 ViewModel을 구성하여 여러 중규모 프로젝트에 적용할 수 있는 **베이스 아키텍처**를 만들어봅니다.


## 주요 패키지 소개
 
이번 포스트는 플러터 개발에 어느정도 익숙해진 분들을 위해 작성되었습니다.

저 역시도 주니어 레벨이고, 이제 막 현업에 입문한 초보 개발자이지만, 플러터 생태계에서 주로 사용되는 패키지들의 역할과 기능을 어느정도 이해하고 있다면 이번 포스트에서 다루는 내용의 필요성을 더 잘 이해하실수 있습니다.

각 주요 패키지들의 기능은 이번 포스트에서 세세하게 다루지 않습니다. 

상술한 듯 이번 포스트는 위 패키지들을 사용하여 베이스 아키텍처를 구성하는 것이 목적이고, 제 설명보다 공식 문서의 내용이 더 정확하고 자세합니다.

이번 문단에서는 베이스 아키텍처에 사용되는 Getx와 Floor의 주요 기능들을 간략하게 소개합니다. 

두 패키지에 대한 이해도가 이미 충분하신 분들이라면, 베이스 아키텍처를 설명하는 문단으로 건너뛰셔도 좋습니다.

### GetX

깃허브 링크: 
**[https://github.com/jonataslaw/getx](https://github.com/jonataslaw/getx)**

GetX는 플러터 생태계에서 가장 많이 사용되는 상태 관리 패키지입니다. 

단순히 상태 관리만을 도와주는 패키지가 아닌, 라우팅, 라이프사이클, 페이지 단위의 모듈 관리 및 데이터 통신 등 플러터라는 **프레임워크 안의 또 다른 프레임워크**로서 작동합니다.


### 상태 관리

이견의 여지가 없는 GetX의 핵심 기능입니다.

플러터의 기본 라이프사이클인 State를 사용하던, [Riverpod](https://github.com/rrousselGit/riverpod)이나 [Provider](https://pub.dev/packages/provider)를 사용하던 상태 관리 패키지의 목적은 동일합니다.

**필요한 데이터만 업데이트하여 불필요한 UI 렌더링을 줄이자**는 것입니다.

이중에서도 GetX는 Obx와 Rx(반응형) 변수를 활용해, 데이터 변경 시 UI가 자동으로 반응하도록 하여 직관적으로 상태 관리를 할 수 있습니다. 베이스 아키텍처에 GetX를 활용하는 가장 큰 이유이기도 합니다.

~~~dart
// 컨트롤러
class CounterController extends GetxController {
  final RxInt count = 0.obs;
  final RxString status = 'Tap + to start'.obs;

  void increment() {
    count.value++;
    _updateStatus();
  }

  void _updateStatus() {
    status.value = count.value % 2 == 0 
      ? 'Even number: ${count.value}'
      : 'Odd number: ${count.value}';
  }
}

// UI. StatefulWidget이 아닌 StatelessWidget을 사용. 정적인 위젯에 갱신이 필요한 부분만 Obx로 감싸기.
class CounterPage extends StatelessWidget {
  final controller = Get.put(CounterController()); // 컨트롤러 등록

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('GetX Counter')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            // count 상태를 관찰
            Obx(() => Text(
              '${controller.count}',
              style: TextStyle(fontSize: 48),
            )),
            
            SizedBox(height: 12),
            
            // status 상태를 관찰
            Obx(() => Text(
              controller.status.value,
              style: TextStyle(fontSize: 16),
            )),
            
            SizedBox(height: 24),
            
            ElevatedButton(
              onPressed: controller.increment,
              child: Icon(Icons.add),
            ),
          ],
        ),
      ),
    );
  }
}
~~~

GetX의 상태 관리는 위와 같이 간단하게 구성할 수 있습니다.

변동될 데이터를 **Rx 변수**로 선언하고, **Obx 위젯**으로 업데이트 될 데이터를 활용하는 View 영역을 감싸 데이터가 변동될 때마다 해당 영역만 자동으로 업데이트 되도록 합니다.

감시할 변수는 **변수명.value** 형태로 접근합니다.

~~~dart
// 데이터 모델
class Artist {
  final RxString name = 'John Mayer'.obs;
  final RxInt age = 45.obs;
  final RxList<String> albums = ['Room for Squares', 'Battle Studies', 'Continuum'].obs;
}

class ArtistController extends GetxController {
  final Rx<Artist> artist = Artist().obs; // 데이터 모델을 Rx 변수로 선언

  // 데이터 업데이트 메서드
  void happyBirthday() {
    artist.value.age.value++;
  }

  void releaseNewAlbum() {
    artist.value.albums.add('Sob Lock');
  }
}
~~~

Rx 변수는 일반적으로 사용되는 기본 타입들(int, String, bool 등)을 포함한 대부분의 타입을 지원하며, 위와 같은 방식으로 커스텀 모델을 포함한 클래스 객체 역시 Rx 변수로 등록할수 있습니다.

### GetxWidget

상태 관리에 대한 대략적인 설명을 마무리했습니다.

헌데, GetX에 대해 생소하신 분들은 상태관리 문단의 코드를 보시면서 처음 보는 부분이 있을 겁니다.

GetX 구조에서는 UI 영역과 비즈니스 로직을 다루는 클래스를 구분하여 페이지를 관리합니다.

이러한 **페이지 모듈**은 대부분의 경우 비슷한 생명주기를 가지며, 라우팅 역시 이 페이지를 기준으로 관리합니다.

페이지 모듈은 GetxView, GetxController, GetxService 클래스로 구성됩니다.

#### GetxController

GetxController는 GetX 구조의 페이지 모듈에서 데이터 영역과 비즈니스 로직을 구성합니다.

~~~ dart
class ArtistController extends GetxController {
  final Rx<Artist> artist = Artist().obs;

  void happyBirthday() {
    artist.value.age.value++;
  }

  void shutDownTheParty() {
    print('Party is over');
  }

  //컨트롤러가 초기화될 때 호출되는 메서드
  @override
  void onInit() {
    super.onInit();
    // 보통 변수의 초기화, 이전 페이지에서 전달된 데이터 처리 등을 이곳에서 구성합니다.
    happyBirthday();
  }

  //컨트롤러가 준비될 때 호출되는 메서드
  @override
  void onReady() {
    super.onReady();
  }

  //컨트롤러가 종료될 때 호출되는 메서드
  @override
  void onClose() {
    super.onClose();
    shutDownTheParty();
  }
}
~~~

이곳에 비즈니스에 필요한 변수(GetX 구조에서는 대부분 Rx 변수)와 메서드를 구성하고, 이를 통해 비즈니스 로직을 구성합니다.

또한, onInit(), onReady(), onClose() 메서드를 오버라이드하여 컨트롤러의 생명주기에 따라 필요한 로직을 구성할 수 있습니다.

생명주기 메소드는 활용하기 나름이지만, 각 메소드들은 보통 다음과 같은 상황에서 호출됩니다:

- onInit(): 컨트롤러가 초기화 될 때. 

특정 페이지에서 컨트롤러를 등록하거나, 컨트롤러와 연결된 페이지로 새로 이동한 경우. 보통 변수의 초기화, 이전 페이지에서 전달된 데이터 처리 등을 이곳에서 구성합니다.

- onReady(): 컨트롤러가 준비될 때(정확히는, UI가 완전히 렌더링 된 후).

 onInit과 다른 점은 페이지의 UI가 모두 렌더링 된 후(1프레임 후)에 호출되는 점입니다.
 
 UI에 초기값을 구성해야 할 때나, UI가 렌더링된 후 특정 비즈니스 로직을 실행해야 한다면 이 메소드를 활용합니다.
- onClose(): 컨트롤러가 종료될 때. 

 주로 등록된 변수나 컨트롤러(TextEditingController, 애니메이션 관련 상태값 등)의 해제, 다시 이전 페이지로 돌아갈 시(Pop 될 때)에 호출되는 비즈니스 로직 등을 이곳에서 구성합니다.


~~~dart
// 컨트롤러를 즉시 메모리에 등록합니다. 연결된 페이지가 Pop 되거나 라우트 스택에서 제거되면 자동으로 해제됩니다.
Get.put(ArtistController());


// 컨트롤러를 등록하지만, find 메소드 등으로 컨트롤러를 처음 호출하기 전까지는 메모리에 올리지 않습니다.
Get.lazyPut(ArtistController());

// 컨트롤러 찾기
Get.find<ArtistController>();

// 컨트롤러 해제
Get.delete<ArtistController>();
~~~

컨트롤러의 의존성을 관리하는 방법은 위와 같습니다.

위 방식들도 많이 사용되지만, 사실 더 간단하게 컨트롤러를 등록하고 해제하는 방법이 있습니다.

바로 GetxView를 통해 컨트롤러를 등록하는 방법입니다.


#### GetxView

GetxView는 StatelessWidget과 같이 별도의 처리 없이는 정적인 상태를 가지는 View 영역의 클래스입니다.

~~~dart
// 컨트롤러를 등록하고, 컨트롤러와 연결된 페이지로 이동할 때 컨트롤러를 자동으로 메모리에 올립니다.
// 반대로 GetView 페이지를 라우팅 스택에서 제거하면 컨트롤러를 자동으로 해제합니다.
class ArtistPage extends GetxView<ArtistController> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Artist Page')),
      body: Center(child: Text('Artist: ${controller.artist.value.name}')),
    );
  }
}
~~~

GetxView의 특징은 위처럼 Get.put() 메소드를 통해 컨트롤러를 등록하는 것이 아닌, 제네릭 타입을 통해 컨트롤러를 등록하는 것입니다.

GetxView를 사용하면 구성한 페이지를 호출하거나 종료될 때 자동으로 컨트롤러가 등록/해제되므로, 보다 직관적이고 편리하게 컨트롤러 의존성을 관리할 수 있습니다.

GetxController를 담을수 있는 StatelessWidget으로 생각하셔도 무방합니다.


#### GetxService

GetxView와 GetxController는 모두 페이지 모듈을 구성하는 클래스입니다.

두 위젯을 통해 페이지 모듈을 구현하는 것만으로도, 비즈니스 로직과 데이터 영역 / View 영역의 책임을 분리하고 효율적인 상태관리가 가능한 구조를 만드실수 있을 겁니다.

하지만 한가지 더 알아두면 유용한 Getx 위젯이 있습니다.

GetxService는 GetxController와 거의 비슷한 기능을 하는 클래스로, 기능 상으로는 큰 차이가 없습니다.

다만 GetxService는 페이지 모듈 단위가 아닌, 전역적으로 사용되는 데이터를 관리하는 데 초점이 맞춰져 있습니다.


~~~dart
class GlobalService extends GetxService {
  final Rx<User> user = User().obs;


  @override
  void onInit() {
    super.onInit();
    getUser();
  }


  Future<void> getUser() async {
    // 예시 구조. 실제 통신을 구현하여 전역적으로 사용될 데이터(유저 등)를 가져옵니다.
    user.value = await getUserFromDatabase();
  }
}
~~~

프로젝트의 규모가 커지면 전역적으로 사용되는 데이터가 생기거나, 범용적으로 사용되는 비즈니스 로직을 관리할 때가 생깁니다. 유저 데이터나 네트워크 통신 등이 있을수 있습니다.

이러한 경우 GetxService를 통해 특정 데이터나 비즈니스 로직을 앱의 전역적인 범위에서 관리할 수 있습니다.

~~~dart
main() {
  runApp(MyApp());

  // initialBinding 옵션을 활용하거나, main 영역에서 할당 가능.
  //Get.put(GlobalService(), permanent: true);
}

class MyApp extends GetxView {
  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      home: HomePage(), 
      initialBinding: BindingsBuilder(() {
        Get.put(GlobalService()); // 서비스 등록
      }),
    );
  }
}
~~~

위와 같은 구조로 GetxService를 앱의 최상위 위젯인 GetMaterialApp에 등록하면, 앱이 시작될 때 자동으로 서비스가 초기화되고, 앱이 종료될때까지 메모리에 유지됩니다.

일반적인 비즈니스 로직 코드는 사용될 때만 메모리에 올리고, 사용이 끝나면 적절히 해제해줘야 하지만, 주기적으로 사용되는 데이터나 로직은 GetxService를 통해 앱 전체의 생명주기와 함께 메모리에 유지하도록 하여 코드를 중앙화하고 메모리 관리를 편리하게 할 수 있습니다.

이를 활용하는 더 자세한 내용은 후술할 베이스 아키텍처 설명 문단에서 다루도록 하겠습니다.

이제 베이스 아키텍처에서 사용하는 GetX의 핵심적인 기능들인 상태 관리와 페이지 모듈 클래스에 대한 설명을 마치겠습니다.

GetX 패키지는 위의 기능들 외에도 스낵바와 같은 유틸 기능이나, 네트워크 통신(http 패키지와 유사) 등 더 다양한 기능을 제공합니다.

이번 포스트에는 베이스 아키텍처에 사용되는 핵심적인 기능들만을 다루기에, 관심이 있다면 공식 문서를 참고해보시면 좋겠습니다.


### Floor

앱의 전반적인 구조에 사용되는 기능들을 다루었으니, Floor에 대한 설명을 시작하겠습니다.

깃허브 링크:
**[https://github.com/tekartik/floor](https://github.com/tekartik/floor)**

Floor는 플러터 생태계에서 가장 많이 사용되는 데이터베이스 패키지는 아니지만, 안드로이드 생태계의 [Room](https://developer.android.com/training/data-storage/room) 라이브러리처럼 유사 관계형 데이터베이스로서 **View Model과 데이터베이스의 역할을 모두 수행**할 수 있습니다.

### Entity

Floor의 데이터베이스는 Entity, DAO, Converter 클래스로 구성됩니다.

Entity는 데이터베이스의 테이블을 구성하는 클래스로, Floor는 Entity 단위로 데이터를 조회하고 저장하며, 저장하지 않더라도 Entity 그 자체를 View Model로서 사용할 수 있습니다.

~~~dart
import 'package:floor/floor.dart';

@entity
class User {
  @primaryKey
  int id;
  String name;
  String email;

  // final 키워드를 사용하는 경우, 필수적으로 초기화 필요.
  // final int id;
  // final String name;
  // final String email;

  // required 키워드를 사용하는 경우, 필수적으로 초기화 필요.
  // required int id;
  // String name;
  // String email;

  // 저의 경우에는, 위처럼 키워드를 사용하지 않고 멤버 변수를 선언한 뒤, 
  // 생성자를 통해 초기화 하는 방식을 사용합니다.
  // 이 경우 기본 값이 등록되어있어 null 체크를 하지 않아도 되므로 편리합니다.
  // 이는 편의성을 중시하는 구조로, 
  // 실제 개발 프로젝트에서는 데이터베이스 테이블에 저장되는 데이터의 형태를 명확하게 정의하는 것이 좋습니다.
  User({
    this.id = 0, 
    this.name = '',
    this.email = '',
  });

  // 복사 생성자. 엔티티를 뷰 모델로서 활용할 때 특정 값만 변동하는 경우(예: 유저 이름 변경),
  // 기존의 다른 값은 유지하고 특정 값만 변경하는 로직에서 유용합니다.
  User.copyWith({
    int? id,
    String? name,
    String? email,
  }) : id = id ?? this.id,
       name = name ?? this.name,
       email = email ?? this.email;

  // json 파싱 과정에서 사용되는 메서드.
  // 데이터베이스에 저장되는 데이터의 형태가 json인 경우나, 
  // 서버와의 통신 과정에서 데이터가 json 형태로 전달되는 경우, 엔티티 변환 시 유용합니다.
  factory User.fromJson(Map<String, dynamic> json) => User(
    id: json['id'],
    name: json['name'],
    email: json['email'],
  );

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'email': email,
  };
}
~~~

Floor의 Entity는 위와 같이 구성됩니다.

일반적인 플러터의 모델 객체를 선언하는 과정과 비슷하지만, Entity는 데이터베이스의 테이블을 구성하는 요소로서 활용하는 방식에 따라 View Model과 데이터베이스의 역할을 모두 수행할 수 있습니다.

~~~dart
  class GlobalManager extends GetxService {
    static GlobalManager get to => Get.find<GlobalManager>();

    // 데이터베이스 접근 변수
    late final AppDatabase db;

    // 데이터베이스에서 데이터를 조회하기 위한 DAO 접근 변수
    UserDao get userDao => db.userDao;

    // Rxable User Entity 변수
    final Rxn<UserEntity> user = Rxn<UserEntity>();

    @override
    void onInit() async {
      super.onInit();
      db = await $FloorAppDatabase.databaseBuilder('app_database.db').build();

      // 데이터베이스에서 데이터를 조회하여 Rxable User Entity 변수에 할당.
      // 이 전에 서버에서 데이터를 조회하는 로직이 있었다면, 이곳에서 데이터를 조회하는 로직을 구성합니다.
      user.value = await userDao.findUserById(1);
    }
  }

  // 유저 데이터를 활용하는 View 영역.
  // 다른 페이지에서 유저 데이터가 변동되어도, 전역적으로 유저 Entity를 관리하기에
  // 유저 데이터가 변동될 때마다 자동으로 업데이트 됩니다.
  class UserPage extends GetxView<UserController> {
    @override
    Widget build(BuildContext context) {
      return Scaffold(
        appBar: AppBar(title: Text('User Page')),
        body: Center(child: Text('User: ${controller.globalManager.user.value?.name}')),
      );
    }
  }
~~~

후술할 베이스 아키텍처를 비롯, 저는 Entity를 뷰 모델로서 활용하기도 하며, 위와 같이 GetxService를 통해 전역적으로 사용되는 데이터를 Entity로 관리하는 경우가 많습니다.

### Converter

Floor 구조에서 위처럼 Entity를 활용하기 위해서는, Converter 클래스를 통해 데이터베이스에 저장되는 데이터의 형태를 정의해야 합니다.

~~~dart
import 'package:floor/floor.dart';

//유저 엔티티 리스트를 데이터베이스에 저장하기 위한 Converter 클래스
@TypeConverter
class UserListConverter extends TypeConverter<List<User>, String> {
  @override
  List<User> decode(String databaseValue) {
    return User.fromJson(jsonDecode(databaseValue));
  }

  @override
  String encode(List<User> value) {
    return value.toString();
  }
}
~~~

~~~dart

enum UserType {
  admin,
  user,
  guest,
}

//유저 타입을 데이터베이스에 저장하기 위한 Converter 클래스
@TypeConverter
class UserTypeConverter extends TypeConverter<UserType, String> {
  @override
  UserType decode(String databaseValue) {
    return UserType.values.firstWhere((e) => e.toString() == databaseValue);
  }

  @override
  String encode(UserType value) {
    return value.toString();
  }
}
~~~

기본적인 타입(String, int, double, DateTime 등)은 지원되지만, enum이나, 엔티티 내부에서 엔티티를 사용하는 등 직접 정의한 타입의 경우 별도의 Converter 클래스를 통해 데이터베이스에 저장되는 데이터의 형태를 정의해야 합니다.

### DAO

이렇게 컨버터와 엔티티를 정의하였다면, 이제 데이터베이스에 접근하기 위한 DAO 클래스를 정의해야 합니다.

~~~dart
import 'package:floor/floor.dart';

@dao
abstract class UserDao {
  @Query('SELECT * FROM user')
  Future<List<User>> findAllUsers();

  //기본 지원하는 메서드들
  @insert
  Future<void> insertUser(User user);

  @delete
  Future<void> deleteUser(User user);

  @update
  Future<void> updateUser(User user);

  // 커스텀 메서드. 유저를 아이디로 조회하는 예시 메서드
  @Query('SELECT * FROM user WHERE id = :id')
  Future<User> findUserById(int id);
}
~~~

DAO는 데이터베이스에 접근하기 위한 메서드들을 정의하는 클래스입니다.

floor에서 기본적으로 지원해주는 CRUD 메서드나, 커스텀 SQL 쿼리를 메서드로 정의할 수 있습니다.

### Database Setting

이제 Floor 데이터베이스를 사용하는데 필요한 모든 요소들을 만들었습니다.

~~~dart
import 'package:floor/floor.dart';

// 데이터베이스 파일 생성 시 자동으로 생성되는 파일을 미리 정의
part 'app_database.g.dart';

@TypeConverters([
  UserListConverter(),
  UserTypeConverter(),
])

@Database(version: 1, entities: [User])
abstract class AppDatabase extends FloorDatabase {
  UserDao get userDao;
}
~~~

실제로 Floor 데이터베이스를 사용하려면, 위와 같이 Database 클래스를 정의합니다.

~~~yaml
# pubspec.yaml
# 플로어 설정을 위한 필수 패키지.
dev_dependencies:
  build_runner: ^2.4.1
  floor: ^2.2.0
  floor_generator: ^2.2.0
~~~

~~~bash
flutter pub run build_runner build
~~~

그리고 build_runner를 통해 데이터베이스 파일을 생성하면, Floor 데이터베이스를 사용할 준비가 완료됩니다.

각 패키지의 주요 기능과 역할에 대한 소개는 여기까지입니다.

저는 어플리케이션을 만드는 과정이 건물을 짓는 과정과 비슷하다고 생각합니다.

GetX는 건물의 튼튼한 **골자와 뼈대**를, Floor는 건물을 사람들이 이용할 수 있도록 **수도와 전기가 드나드는 통로**를 이룹니다.

제대로 된 구조를 만들면, 멋진 외장과 인테리어를 꾸미는 일도 수월해질 겁니다.

이제 두 패키지가 어떤 역할을 하는지 이해하셨다면, 실제로 베이스 아키텍처를 구성해봅시다.

## 베이스 아키텍처

본격적인 베이스 아키텍처 설명에 앞서, 제가 소개하는 구조는 현업에서 프로젝트를 진행하며 저와 팀원 분들의 작업 환경에 맞추어 설계되어 당연하게도 모든 프로젝트에 완벽하게 기능하는 구조는 아닙니다.

- 10명 이하의 팀원으로 구성되어, 10개에서 30개 이하의 페이지를 구성하는 **소, 중규모 프로젝트 대상**
- 디자이너와 협업하며, 대다수의 페이지가 특정 스타일 가이드를 따르는 프로젝트
- 비즈니스 로직과 데이터베이스 통신이 필요하나, **앱에 저장되는 영역과 View Model 영역이 혼합되어 있는 프로젝트**
- **GetX 구조를 기반**으로 빠른 생산성이 필요한 프로젝트
- 유저 데이터 등 **전역적으로 활용하는 데이터가 있는 프로젝트(플랫폼 앱과 같은)**
- 대다수의 페이지 모듈이 비슷한 생명주기와 구조를 가지는 프로젝트

다만 위과 같은 요구사항에 맞추어 개발되었기에, 저희와 목적이 비슷한 분들에게는 참고가 될 수 있습니다.

### Base Module

베이스 아키텍처는 비슷한 구조의 여러 뷰를 사용할 때 각 페이지에 일관성과 개발 편의성을 제공하는 것이 주 목적입니다.

따라서, 일관성있는 페이지 모듈 구조를 위해 **BaseView**와 **BaseController**를 사용합니다.

~~~dart
//BaseView는 모든 뷰의 기본 구조를 정의하는 클래스입니다.
//GetxView를 상속받아 모든 뷰에서 사용할 수 있도록 합니다.
abstract class BaseView<Controller extends BaseController> extends GetView<Controller> {

  final GlobalKey<ScaffoldState> globalKey = GlobalKey<ScaffoldState>();

  @override
  Widget build(BuildContext context) {
    // 상세한 UI 구성은 후술...
    return Scaffold(
      appBar: AppBar(title: Text('Base Module')),
      body: Center(child: Text('Base Module')),
    );
  }
}
~~~

~~~dart
//BaseController는 모든 컨트롤러의 기본 구조를 정의하는 클래스입니다.
//GetxController를 상속받아 모든 컨트롤러에서 사용할 수 있도록 합니다.
class BaseController extends GetxController with GetSingleTickerProviderStateMixin {
  // 상세한 비즈니스 로직은 후술...
  @override
  void onInit() {
    super.onInit();
  }
}
~~~

BaseView는 GetxView를, BaseController는 GetxController를 상속받아 모든 페이지에서 Getx 구조를 일관성 있게 사용할 수 있습니다.

~~~
lib/
  pages/
    home/
      home_view.dart # BaseView
      home_controller.dart # BaseController
    login/
      login_view.dart # BaseView
      login_controller.dart # BaseController
    my_page/
      my_page_view.dart # BaseView
      my_page_controller.dart # BaseController
~~~

상술한 GetX 패키지 설명에서 의도대로, BaseView는 UI 구성을, BaseController는 비즈니스 로직을 구성하는 책임을 맡아 두 모듈이 하나의 페이지 구조를 이룹니다.

#### BaseController

베이스 모듈 구조가 왜 필요한지 말씀드렸으니, BaseController부터 사용 예시를 보여드리겠습니다.

~~~dart
import 'package:get/get.dart';
import 'package:logger/logger.dart';

enum PageState {
  defaultState,
  loading,
  success,
  failed,
  updated,
  created,
  noInternet,
  message,
  unAuthorized,
}

enum DevicePlatform {
  android,
  ios,
  ipadOs,
  web,
  desktop,
}

class BaseController extends GetxController with GetSingleTickerProviderStateMixin {

  final GlobalManager global; //여러 곳에서 쓰이므로, 명칭 단순화. (제 취향입니다)
  //싱글톤 패턴으로 생성된 GetxService(Manager)를 주입받습니다.
  BaseController() : global = Get.find<GlobalManager>();

  //만약 여러개의 매니저를 사용한다면, 파라미터로 추가합니다.
  // final NetWorkManager network;
  // final UserManager user;
  //BaseController() : 
  //  global = Get.find<GlobalManager>(), 
  //  network = Get.find<NetworkManager>(), 
  //  user = Get.find<UserManager>();
  //  ...


  //Logger 패키지 접근.
  final logger = Logger();

  //크로스 플랫폼 분기처리를 위한 변수.
  Rx<DevicePlatform> devicePlatform = DevicePlatform.android.obs;

  //페이지 상태 변수.
  Rx<PageState> pageState = PageState.defaultState.obs;

  //페이지 상태 변수.
  RxBool isRefresh = false.obs;

    // // 키보드 보일때 안보일떄 화면 처리를 위해
  late StreamSubscription keyboardSubscription;
  final RxBool _isKeyboardVisible = false.obs; // 키보드 보임여부
  bool get isKeyboardVisible => _isKeyboardVisible.value;

  // 키보드 상태에 따른 자동 레이아웃 변경 여부
  final RxBool _isKeyboardAutoLayout = true.obs;
  bool get isKeyboardAutoLayout => _isKeyboardAutoLayout.value;
  set isKeyboardAutoLayout(bool val) => _isKeyboardAutoLayout.value = val;

  //로딩 상태. 로딩 유틸리티와 조합.
  bool isShowLoading = false;
  LoadingType loadingType = LoadingType.normal;

    bool refreshPage(bool refresh) => _refreshController(refresh);

  //Controls page state
  final _pageSateController = PageState.defaultState.obs;

  PageState get pageState => _pageSateController.value;

  PageState updatePageState(PageState state) => _pageSateController(state);

  PageState resetPageState() => _pageSateController(PageState.defaultState);

  PageState showLoading({LoadingType type = LoadingType.normal}) {
    if (!isShowLoading) {
      isShowLoading = true;
      loadingType = type;
      // WidgetsBinding.instance.addPostFrameCallback((_) {
      updatePageState(PageState.loading);
      // });

      return PageState.loading;
    }

    //page state return.
    return pageState;
  }

  PageState hideLoading() {
    if (isShowLoading) {
      WidgetsBinding.instance.addPostFrameCallback((_) {
        isShowLoading = false;
        resetPageState();
      });

      return PageState.defaultState;
    }

    return pageState;
  }


  //뒤로가기 버튼 클릭 시 호출되는 메서드.
  Future<bool> goBack() {
    Get.back();

    return Future.value(true);
  }

  bool isCanSwipeBack() {
    if (!Platform.isAndroid) {
      /// iOS
      return true;
    }

    /// Android
    return false;
  }


  @override
  void onInit() {
    super.onInit();
  }
}
~~~

베이스 아키텍처의 사용 방법은 프로젝트의 규모가 방향성에 따라 갈리겠지만, 일반적으로 BaseController는 프로젝트의 모든 페이지가 공통적으로 사용하는 비즈니스 로직을 담고 있습니다.

- 페이지 상태 관리
- 크로스 플랫폼 분기처리
- 네트워크 관리
- 자주 사용되는 유틸리티 함수 접근(Logger 등)
- 데이터베이스 관리
- 디바이스 정보 관리
- 키보드 관리
- 로딩 관리
- 메시지 관리
- 유저 관리

저는 보통 위의 기능들을 모두 포함하는 클래스를 만들어 사용합니다.

특정 역할을 수행하면서, 앱 전역에서 사용되는 비즈니스 로직의 코드가 많아지면 위의 예시처럼 **Manager 클래스**를 만들어 사용합니다.


~~~dart
class GlobalManager extends GetxService {
  //접근 편의를 위해 정적 메서드를 정의합니다.
  static GlobalManager get to => Get.find<GlobalManager>();
  
  // 전역적으로 사용되는 비즈니스 로직 작성.
  // 저는 보통 GlobalManager에 Floor 데이터베이스를 접근하는 코드를 작성합니다.
  late final AppDatabase db;

  //앱 내 모든 DAO를 접근하기 위한 변수
  UserDao get userDao => db.userDao;

  Future<void> init() async {
    db = await $FloorAppDatabase.databaseBuilder('app_database.db').build();
  }

  @override
  void onInit() {
    super.onInit();
    init();
  }

  // 종료될 일이 없으므로, onClose는 작성하지 않습니다.
}
~~~

Manager 클래스는 상술한 GetxService를 상속한 클래스로, 앱 전역에서 사용되는 특정 비즈니스 로직을 모아둡니다.

앱 데이터베이스 관리나, 네트워크 통신 관련 메소드를 모아둔 코드 등을 Manager로 작성한 후, BaseController에서 접근하도록 하면 코드의 응집도를 높일 수 있습니다.

페이지 단위에서 공통적으로 사용되는 비즈니스 로직은 BaseController에, 앱 전역에서 공통적으로 사용되나 특정 기능을 책임지는 코드들을 Manager로 작성하여 BaseController에 추가하는 방식입니다.


~~~dart
class MyPageController extends BaseController {
  @override
  void onInit() {
    super.onInit();
  }

  Future<void> updateUser() async {
    // 글로벌 매니저에 선언된 userDao를 접근하여, DAO가 지닌 메소드를 호출.
    // 글로벌 매니저에 정의된 user 변수를 업데이트. 앱 전역에 유저의 상태가 업데이트됩니다.
    global.user = await global.userDao.updateUser(user);
  }
}
~~~

실제 사용 예시는 위와 같을수 있습니다.


#### BaseView

다음은 BaseView입니다.

**BaseView**는 **BaseController**를 상속받아 모든 페이지에 사용되는 UI 영역의 설정을 담당합니다.

~~~dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

class BaseView<Controller extends BaseController> extends GetView<Controller> {
  final GlobalKey<ScaffoldState> globalKey = GlobalKey<ScaffoldState>();

  // AppLocalizations get appLocalization => AppLocalizations.of(Get.context!)!;

  Widget body(BuildContext context);

  Widget? appBar(BuildContext context);

  Widget? bottomButton(BuildContext context) {
    return null;
  }

  // 페이지 설정에 관한 매개변수
  final bool isBackgroundImageEnable;
  final bool resizeToAvoidBottomInset;
  final bool isCanPop;
  //isCanPop이 false라면 호출 가능한 함수입니다.
  final Function()? onWillPop;
  final Function()? onAnyTap;

  // 사이즈 설정
  double appBarHeight = 48.s;
  double bottomButtonHeight = 72.s;
  double bodyHeight = Get.height;

  BaseView({
    super.key,
    this.isBackgroundImageEnable = false,
    this.resizeToAvoidBottomInset = true,
    this.isCanPop = true,
    this.onWillPop,
    this.onAnyTap,
  });

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        AnnotatedRegion<SystemUiOverlayStyle>(
          value: const SystemUiOverlayStyle(
            statusBarColor: Colors.white,
            statusBarIconBrightness: Brightness.dark,
            systemStatusBarContrastEnforced: false,
          ),
          child: Material(
            color: Colors.transparent,
            child: WillPopScope(
              key: key,
              onWillPop: isCanPop? ()=> controller.goBack() : () async => onWillPop!(),
              child: Scaffold(
                resizeToAvoidBottomInset: resizeToAvoidBottomInset,
                backgroundColor: pageBackgroundColor(),
                key: globalKey,
                floatingActionButton: floatingActionButton(),
                body: Obx(() => pageContent(context)),
                drawer: drawer(),
              ),
            ),
          ),
        ),
        Obx(
          () => controller.pageState == PageState.loading
              ? _showLoading(controller.loadingType) : Container(),
        ),
      ],
    );
  }

  Widget pageContent(BuildContext context) {

    Widget? topAppBar = appBar(context);
    Widget? bottomBtn = bottomButton(context);

    if(!controller.isKeyboardAutoLayout && MediaQuery.of(context).viewInsets.bottom > 0) {
      bottomBtn = null;
    }

    // 상단영역 크기
    double topHeight = AppUIConfig.safeAreaTop;
    if(topAppBar != null) {
      topHeight = topHeight + appBarHeight;
    }
    // 하단영역 크기
    double botHeight = AppUIConfig.safeAreaBottom;
    if(bottomBtn != null) {
      botHeight = botHeight + bottomButtonHeight;
    }

    // 컨텐츠영역 Body 영역 구하기
    bodyHeight = Get.height - topHeight - botHeight;


    return InkWell(
      onTap: () {
        onAnyTap;
        FocusScope.of(context).unfocus();
      },
      splashColor: Colors.transparent,
      highlightColor: Colors.transparent,
      child: Stack(
        alignment: Alignment.topCenter,
        children: [
          Container(
            child: topAppBar ?? Container(),
          ),
          Stack(
            alignment: Alignment.bottomCenter,
            children: [
              Container(
                  color: Palette.bgWhite,
                  margin: EdgeInsets.only(top: topHeight, bottom: botHeight),
                  height: bodyHeight,
                  child: body(context)
              ),
              Container(
                color: Palette.bgWhite,
                height: botHeight,
                child: bottomBtn ?? Container(),
              ),
            ],
          ),
        ],
      )
    );
  }

  Color pageBackgroundColor() {
    return Colors.transparent;
  }
~~~

BaseView 역시 BaseController와 마찬가지로 필요로 하는 앱의 규모나 방향성에 따라 기능을 다르게 설정할 수 있지만, 저의 경우에는 다음과 같은 기능들로 구성하여 사용하고 있습니다:

- 페이지 상태 관리. Loading 상태를 예로 들면 Loading 상태 돌입 시 타입에 따라 화면에 보이는 로딩을 설정합니다. (전체 화면 로딩, 중앙 등)
- AppBar, body, bottomButton 등의 앱에서 자주 사용되는 영역 설정. AppBar와 bottomButton은 null을 허용하여 필요에 따라 오버라이드 할 수 있도록 합니다.
- 키보드 상태에 따른 자동 레이아웃 변경 여부. 키보드가 보일때 안보일때 화면 처리를 위해 사용합니다.
- 스테이터스 바 컬러, 기본 배경 컬러 및 패딩으로 디자이너와 협업할 경우의 앱 스타일 설정.


~~~dart
//BaseController를 상속받은 HomeController가 주입된 BaseView
class HomeView extends BaseView<HomeController> {
  // null 허용. 앱바가 존재하지 않는 뷰를 대비
  @override
  Widget? appBar(BuildContext context) {
    // 앱바나 바텀버튼 등은 컴포넌트로 분리하여 커스텀 위젯을 사용하면 편리합니다.
    return CAppBar(title: Text('Home'), type: AppBarType.back);
  }

  @override
  Widget body(BuildContext context) {
    return Column(
      // 페이지 구성 위젯 등... 예시.
      children: [
        Text('Home'),
        Banner(),
        UserCircle(),
      ],
    );
  }

  // null 허용. 바텀버튼이 존재하지 않는 뷰를 대비
  @override
  Widget? bottomButton(BuildContext context) {
    // 앱바나 바텀버튼은 컴포넌트로 분리하여 커스텀 위젯을 사용하면 편리합니다.
    return CBottomButton(
      onPressed: () {
        controller.onTapBottomButton();
      },
      child: Text('다음'),
    );
  }


  // 페이지 구성 위젯 등... 예시.
  Widget Banner() {
    return Container(
      color: Palette.bgWhite,
      //상태 변화 감지 예시.
      child: Obx(() => Text(controller.bannerText.value)),
    );
  }

  Widget UserCircle() {
    return Container(
      color: Palette.bgWhite,
      child: Text('UserCircle'),
    );
  }
}
~~~

사용 예시는 위와 같을수 있습니다.

사용 여부에 따라 BaseView에 등록된 Appbar, BottomButton 등을 오버라이드 하면, BaseView에서 지정된 패딩 값과 위치에 따라 화면이 구성됩니다.

body 영역에는 페이지 구성 위젯들을 작성하면 됩니다.

BaseView는 GetX 구조와 유사하게 BaseController를 제네릭 타입으로 주입받으므로, 기존 GetX 구조와 동일한 상태관리를 사용할 수 있습니다.

### 마치며

이번 포스트에서는 제가 프로젝트에서 사용하는 베이스 아키텍처를 소개해드렸습니다.

저는 이번으로 세 번째 업무 프로젝트를 모두 베이스 아키텍처를 사용하여 개발하고 있습니다.

처음에는 이 복잡한 구조를 이해하긴 커녕 필요성조차 잘 느끼지 못했지만, 이제는 나름 스스로 작업 속도도 빨라지고, 전보다 집에 빨리 들어가는 것 같습니다.

본문은 장황하게 적었지만 사실 지금껏 설명드린 베이스 아키텍처 역시 완전하지 않습니다.

아키텍처는 프로젝트의 규모와 방향성에 따라 필요한 기술과 구조가 달라질 수도 있고, 이미 포스트를 작성하는 이 순간에도 기능 요구사항을 완벽하게 충족하지 못해 몇 번의 업데이트가 추가된 상황입니다.

하지만 구조를 정의하고 학습하는 과정에서 배운 코드의 통일성과 재사용성은 회사에서의 업무와 제 개인 프로젝트 모두에 큰 도움이 되고 있습니다.

꼭 위에서 소개한 베이스 아키텍처일 필요는 없습니다.

여러분도 재사용성과 통일성을 추구하는 자신만의 아키텍처를 만들어 보세요. 똑똑하게 게을러지는 방법입니다.

부족하고 긴 글을 끝까지 읽어주셔서 감사합니다.
