```java
// 특정 객체에 대해 dynamic sql을 적용해보자

public class Student{
	private Long id;
	private String name;
	private Integer age;
}

public final class StudentDynamicSqlSupport{
	
	public static final Student student = new Student(); // <- 이 생성자를 하위에 만들어주자
	
	public static final class Student extends SqlTable{
	
	// 자료형에 따른 SqlColumn변수로 column(컬럼명) 을 지정한다.
		public final SqlColumn<Long>  id = column("id");
		public final SqlColumn<String> name = column("name");
		public final SqlColumn<Integer> age = column("age");
	}
		
	public static final SqlColumn<Long> = student.id; // 위에 설정했던 값을 final 변수로 활용하기 위해 넣어주기
	public static final SqlColumn<String> = student.name; // 위에 설정했던 값을 final 변수로 활용하기 위해 넣어주기
	public static final SqlColumn<Integer> = student.age; // 위에 설정했던 값을 final 변수로 활용하기 위해 넣어주기

}
```

Mapper을 통해 dynamic support 값들을 활용해 쿼리를 작성해보자

## create

```java

default String insertStudent(Student student){
		MyBatis3Utils.insert(this::insert, student,
		StudentDynamicSqlSupport.student,
		c->c.map(name).toProperty("name") // -> class의 어떤 값으로 매핑해줄지 (Java side)
		.map(age).toProperty("age"));
		return student.id;
	)
}
```

## Update

```java
// update를 하기 위해선 baseEntityAuditingListener를 설정해주어야 함.
public class Student{
	private Long id;
	private String name;
	private Integer age;
	
	public void touchForUpdate() {
		BaseEntityAuditingListener listener = new BaseEntityAuditingLisenter();
		listener.touchForUpdate(this);
	}
	
}
private int update(UpdateDSLCompleter completer){
	return MyBatis3Utils.update(this::update, student, completer);
}

default void updateById(Student student){
		return update(c-> c.set(name).equalTo(student.getname())
		.where(id, isEqualTo(student.getId()));
	)
}
```

## READ

```java

// 조인이나, 여러 다른 컬럼을 조합해서 사용한다고 하면 그에 맞는 매핑될 테이블을 작성해주기도 한다.

BasicColumn[] selectList = BasicColumn.columnList(id, name, age); // id, name, age는 final static으로 설정했던 dynamic support의 column을 가져온다.

// SelectProvider : select 쿼리문 결과에 대해 ResultMap 처럼 매핑 해줌
@SelectProvider(type = SqlProvicderAdapter.class, method = "select")
@Results(id = "resultmapid" value = {
	@Result(column = "id", property = "id", id = true)
	@Result(column = "name", property = "name")
	@Result(column = "age", property = "age")
})
List<Student> selectMany(SelectStatementProvider selectStatuementProvider);

// 진짜 select 하고 싶은 쿼리 함수명 맞춰 작성

default List<Student> findStudnetByAge(Integer age, Pageable pageable){
	QueryExpressionDSL<SelectModel>.QueryExpressionWhereBuilder 
	queryBuilder	= SqlBuilder.select(
		selectList
	).from(studnet).where(age, equalTo(age));
	
	return selectMany(queryBuilder.build().render(RenderingStrategies.MYBATIS3))
}
```
