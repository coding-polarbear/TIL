# SQLiteOpenHelper 클래스
* `SQLiteOpenHelper` 클래스는 `SQLite`에 접근하는 클래스
* `SQL` 명령을 실행하고 DB를 관리하는 메소드를 가지고 있음.
* SQLite를 사용하기 위해서는 꼭 거쳐야 하는 클래스이지만 실제 앱에서 `SQLiteDatabase`를 직접 생성하고 접근해서 사용하는 경우는 드믐.
* `SQLiteOpenHelper`를 상속해서 사용하는데, `SQLite`에 접근할 때 `SQLiteOpenHelper`에서 DB 생성이나 DB 버전관리를 알아서 해줌.
* 일반적으로 앱은 업데이트됨에 따라, DB에 테이블이 추가되거나 칼럼이 변경되고 앱에 필요한 기본 데이터가 추가 되기도 함.
  * 버전관리가 필수적
  * `SQLiteDatabse`를 직접 사용하지 않고 반드시 `SQLiteOpenHelper`를 사용해야 함.
* `SQLiteOpenHelper`는 추상클래스이면서 일종의 템플릿 메소드 패턴을 만들어 놓은 것.
* 이 클래스를 상속해 `onCreate()`와 `onUpgrade()`메소드를 구현하면 됨.

**SQLiteOpenHelper를 상속한 DB 헬퍼**
```java
public class DatabaseHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "loader_throttle.db";
    private static final int DATABASE_VERSION = 2;

    DatabseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.executeSQL("CREATE TABLE " + MainTable.TABLE_NAME + " ("
            + MainTable._ID + " INTEGER PRIMARY KEY,"
            + MainTable.COLUMN_NAME_DATA + " TEXT" + ");");
        ...
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        for(int i = oldVersion+1; i <= newVersion; i++) {
            processUpgrade(db, i);
        }
    }
}
```

## DB마다 별도의 DB 헬퍼가 필요
* 여러개의 DB를 사용하려면 여러개의 DB 헬퍼가 필요함.
* 기본적으로 DB 하나당 DB 헬퍼가 1개 필요함.
* 가능하면 DB를 하나로 사용하는 것이 좋지만, DB 락 문제를 효과적으로 대응하기 위해서 DB를 분리하기도 함. (읽기 전용 데이터를 위한 DB, 읽기 + 쓰기를 위한 DB)
* 테이블과 데이터 구조가 동일한 여러 DB가 있다면 생성자에서 DB 파일 명을 상수로 만들지 않고 동적으로 전달하면 됨.

## Cursor의 구현체는 주로 SQLiteCursor 사용
* `SQLiteOpenHelper`의 생성자의 세번째 파라미터는 Cursor
* 쿼리 결과로 Cursor의 구현체인 `SQLiteCursor`를 싸용한다면 `null`을 인자로 넣으면 됨.
* 대부분 `SQLiteCursor`를 사용하기 때문에 보통 null이 들어감.
* `Cursor` 구현을 생성하는 팩토리인 `SQLiteDatabase.CursorFactory`를 새로 만들어서 파라미터에 전달 할 수 있지만 필요한 경우는 사실상 없음.

## DB 생성 시점
* DB의 생성 시점은 `SQLiteOpenHelper`의 생성자가 아님.
* 실제로 DB 열기 / 생성 시점은 `SQLiteOpenHelper`의 `getReadableDatabase()`나 `getWritableDatabase()`를 호출하는 시점.
* `SQLiteOpenHelper`는 `SQLiteDatabase` 인스턴스를 1개 가지고 있는데 이 인스턴스가 앞에 이미 생성되었으면 그것을 사용함.
* 인스턴스가 생성된게 없을 경우에는 인스턴스를 새로 생성하고서는 `onCreate()`나 `onUpgrade()` 메소드를 실행함.

## DB 버전 업그레이드
* DB 테이블 변경 시에는 `DatabaseHelper` 생성자에 새로운 버전을 전달하고, `SQLiteOpenHelper`의 `onUpgrade()` 메소드에 변경 내용을 적용하면 됨.
* 이때 표준 패턴은 for 문을 적용해서 `oldVersion`과 `newVersion` 범위 사이에 DB 버전에 했던 작업들을 처리함.

## onCreate()와 onUpgrade() 메소드는 둘 중의 하나만 실행
* `onCreate()` 메소드에서는 최신 DB 스키마와 데이터를 반영해야함.
* `onCreate()` 메소드는 앱이 처음 DB를 생성할 때 호출됨.
* 초기 DB 생성 이후 중간에 앱을 업데이트하지 않았다가 최신버전으로 업데이트했다면 `onUpgrade()` 메소드가 호출됨.
* 앱을 새로 설치했다면 `onCreate()` 메소드만 호출되고 `onUpgrade()`는 실행되지 않음
* `onCreate()`와 `onUpgrade()` 메소드는 둘 중에 하나만 호출됨.
* 따라서 최신 DB 스키마와 데이터를 반영할 수 있도록 버전이 올라갈 때 마다 `onCreate()` 메소드를 수정해야함.

## DB 스키마 변경 시 반드시 DB 버전 업그레이드
* DB 스키마가 변경될 때에는 반드시 DB 버전 업그레이드가 필요함.

## onCreate()와 onUpgrade() 메소드는 트랜잭션으로 이미 감싸져있음.
* `onCreate()`와 `onUpgrade()` 메소드에서는 테이블을 생성 / 수정하는 것 뿐만 아니라 많은 양의 기본 데이터를 추가하거나 업데이트하는 작업도 필요함.
* 이때 데이터 추가는 건당으로는 시간이 얼마 안걸리지만 데이터 개수가 천건, 만건이 된다면 속도가 크게 떨어짐.
* 그래서 속도를 높이기 위해 트랜잭션을 써야 함.
* `SQLiteOpenHelper`에서 이미 `onCreate()`와 `onUpgrade()` 메소드를 1개의 트랜잭션으로 감쌌기 때문에 또 다시 트랜잭션을 고려할 필요가 없음.

## 메모리 DB
* DATABASE_NAME 값 대신 null을 넣으면 파일 DB가 아닌 메모리 DB가 만들어짐.
* 파일 DB도 속도가 느리지는 않지만 메모리 DB가 훨씬 빠름.
* 메모리 DB는 프로세스가 종료되거나 DB가 닫히면 사라져버리는 휘발성 DB이므로 일종의 캐시 용도로 사용하는 것이 좋음.
* 코드 상에서 캐시 자료구조를 만들어도 되지만 굳이 메모리 DB를 쓰는 이유는 그 안에서 쿼리를 실행할 수 있기 대문.
* 메모리 DB에서는 버전 업그레이드가 의미가 없으므로 `version`은 신경쓸 필요가 없음.

## DB 헬퍼는 싱글톤으로 유지
* DB 헬퍼는 앱 전체에 걸쳐 단일 인스턴스를 가지고 있어야 DB 락 문제에서 자유로움.
* 그래서 일반적으로는 싱글톤 패턴을 만들어서 사용함.

```java
public class DatabaseHelper extends SQLiteOpenHelper {
    private static DatabaseHelper instance;

    public static synchronized DatabaseHelper getInstance(Context context) {
        if(instance == null)
            instance = new DatabaseHelper(context.getApplicationContext());
        return instance;
    }

    private DatabaseHelper(Context context) {
        ...
    }
}
```

## close 메소드는 거의 사용하지 않음.
* `close()` 메소드는 호출할 필요가 거의 없음.
* `SQLiteOpenHelper`의 `close()` 메소드는 `SQLiteDatabse` 인스턴스의 `close()` 메소드를 호출하고, `SQLiteDatabase` 인스턴스를 null로 만듬.
* `SQLiteDatabase`를 닫지 않고 인스턴스를 계속 사용해도 문제가 없음.
* `close()` 메소드를 사용하지 않는 또 다른 이유는 `close()` 실행 시점 때문에 문제가 발생할 수 있기 때문.

## onConfigure()와 onOpen() 메소드로 DB 기능 변경
* DB 기능을 변경할 수 있는 메소드에는 `onConfigure(SQLiteDatabase db)` 메소드와 `onOpen(SQLiteDatabase db)` 메소드가 있음.
* `onConfigure()` 메소드는 `SQLiteDatabase` 생성 / 열기 이후, `onCreate()`와 `onUpgrade()` 메소드 전에 실행되는 것으로 WAL(write-ahead logging)이나 외래키(foreign key) 지원 같은 기능을 활성화 할 수 있음.
* `onOpen()` 메소드는 `onCreate()`와 `onUpgrae()` 이후에 DB 연결 설정을 변경할 때 사용함.