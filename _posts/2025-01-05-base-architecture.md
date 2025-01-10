---
layout: post
title: "Flutter 베이스 아키텍처로 일 쉽게 만들기"
date: 2025-01-05
tags: [Flutter, Architecture, GetX, Floor]
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

GetX에서는 모든 페이지를 위한 라우팅 기능을 제공하는 대신, 페이지 단위의 모듈을 관리하는 방식을 사용합니다.

이러한 모듈은 대부분의 경우 페이지와 비슷한 생명주기를 가지며, 모듈 단위로 라우팅을 관리하는 방식을 사용합니다.

이러한 모듈을 위한 클래스는 GetxView, GetxController, GetxService 클래스로 구성됩니다.

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

 컨트롤러가 초기화 된 후 호출되는 메서드. 보통 데이터 초기화 등을 이곳에서 구성합니다.
- onClose(): 컨트롤러가 종료될 때. 

등록된 변수의 해제, 이전 페이지로 돌아갈 시에 호출되는 비즈니스 로직 등을 이곳에서 구성합니다.


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

위 방식들도 많이 사용되지만, 사실 더 간단하게 컨트롤렁를 등록하고 해제하는 방법이 있습니다.

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

GetxView의 특징은 위처럼 Get.put() 메소드를 통해 컨트롤러를 등록하는 것이 아닌, 클래스 자체를 통해 컨트롤러를 등록하는 것입니다.

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

프로젝트의 규모가 커지면 전역적으로 사용되는 데이터를 관리하는 것이 필요해집니다.

이러한 경우 GetxService를 통해 전역적으로 사용되는 데이터를 관리할 수 있습니다.

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

위와 같은 구조로 GetxService를 앱이 시작되는 부분의 GetMaterialApp 위젯에 등록하면, 앱이 시작될 때 자동으로 서비스가 초기화되고, 앱이 종료될때까지 메모리에 유지됩니다.

비즈니스 로직을 관리하는 코드는 사용될 때만 메모리에 올리고, 사용이 끝나면 적절히 해제해줘야 하지만, 몇몇 데이터 관리 로직(유저 데이터 등)은 GetxService를 통해 앱 전체의 생명주기와 함께 메모리에 유지하도록 하여 메모리 관리를 편리하게 할 수 있습니다.

이에 대한 더 자세한 내용은 후술할 베이스 아키텍처 설명 문단에서 다루도록 하겠습니다.

이제 베이스 아키텍처에서 사용하는 GetX의 핵심적인 기능들인 상태 관리와 페이지 모듈 클래스에 대한 설명을 마치겠습니다.

GetX 패키지는 위의 기능들 외에도 스낵바와 같은 유틸 기능이나, 네트워크 통신 등 더 다양한 기능들을 제공하지만, 베이스 아키텍처에 핵심적으로 사용되는 기능 외에는 이번 포스트에서 다루지 않겠습니다.


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

floor에서 기본적으로 지원해주는 CRUD 메서드나, SQL 쿼리를 커스텀 클래스로 정의할 수 있습니다.

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

사용할 컨버터와 DAO, 엔티티를 정의하고, build_runner를 통해 데이터베이스 파일을 생성합니다.

~~~bash
flutter pub run build_runner build
~~~


각 패키지의 주요 기능과 역할에 대한 소개는 여기까지입니다.

저는 어플리케이션을 만드는 과정이 건물을 짓는 과정과 비슷하다고 생각합니다.

GetX는 건물의 튼튼한 **골자와 뼈대**를, Floor는 건물을 사람들이 이용할 수 있도록 **수도와 전기가 드나드는 통로**를 이룹니다.

제대로 된 구조를 만들면, 멋진 외장과 인테리어를 꾸미는 일도 수월해질 겁니다.

이제 두 패키지가 어떤 역할을 하는지 이해하셨다면, 실제로 베이스 아키텍처를 구성해봅시다.

## 베이스 아키텍처

본격적인 베이스 아키텍처 설명에 앞서, 제가 소개하는 구조는 현업에서 프로젝트를 진행하며 저와 팀원 분들의 작업 환경에 맞추어 설계되어 당연하게도 모든 프로젝트에 완벽하게 기능하는 구조는 아닙니다.

다만 다음과 같은 요구사항에 맞추어 개발되었기에, 저희와 목적이 비슷한 분들에게는 참고가 될 수 있습니다.

- 10명 이하의 팀원으로 구성되어, 10개에서 30개 이하의 페이지를 구성하는 **소, 중규모 프로젝트 대상**
- 디자이너와 협업하며, 대다수의 페이지가 특정 스타일 가이드를 따르는 프로젝트
- 비즈니스 로직과 데이터베이스 통신이 필요하나, **앱에 저장되는 영역과 View Model 영역이 혼합되어 있는 프로젝트**
- **GetX 구조를 기반**으로 빠른 생산성이 필요한 프로젝트
- 유저 데이터 등 **전역적으로 활용하는 데이터가 있는 프로젝트(플랫폼 앱과 같은)**
- 대다수의 페이지 모듈이 비슷한 생명주기와 구조를 가지는 프로젝트

### Base Module

### Manager

### UI

### Repository