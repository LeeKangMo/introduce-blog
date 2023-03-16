---
title: MSSQL Unicode 문제
categories: [develop]
comments: true
---

### ** MS SQL Server에서 JPA NativeQuery 사용시 유니코드(Unicode) 문제

필자의 회사는 MSSQL + Mybatis 를 사용한다. MSSQL 같은경우 기본 String Type 이 nvarchar로 

들어가는데 만약 DB Index가 varchar 형이면 해당 인덱스를 타지않아, 소프트웨어 속도에

이슈가 생길수있다. [참조링크](https://engineering-skcc.github.io/ms%20sql%20server/SqlServer-nvarchar/) 

해당 문제를 해결하기위해 sendStringParametersAsUnicode=false 를 추가하였는데, 추가를하니

이번엔 유니코드 문자가 MSSQL DB INSERT시 ? 로 저장되는 현상이 일어났다.. 어쩌라는것이여

생각보다 많은시간을 찾아본것같다. jdbcType, convert, cast 등 안해본것이 없었다.

방법이 달랐다라고 느끼고 MyBatis 를 찾아보니 공식문서를 설명해주신 블로거를 찾았다.

[MyBatis 공식문서링크](https://goodgid.github.io/MyBatis-Docs-typeHandlers/)

이곳에서 정답을 찾았다. 한줄기광명 같은 블로그. 다시한번 감사드린다.

**지원하지 않거나 비표준인 타입에 대해서는
typeHandler를 만들 수 있다.
또한 기존의 typeHandler를 override 할 수도 있다.
override 하는 방법은 2가지가 있다.
Interface(org.apache.ibatis.type.TypeHandler)를 구현한다.
org.apache.ibatis.type.BaseTypeHandler를 extend 한다.
override를 하였으면
optionally하게 그것을 JDBC 타입으로 매핑해준다.**

바로 typeHandler를 상속받아 NVarchar 를 저장할수있는 Class 를 만들어주고!


        @MappedJdbcTypes(JdbcType.NVARCHAR)
        public class PreparedSatementNVarchar extends BaseTypeHandler {``

			@Override
			public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
				ps.setNString(i, (String) parameter);
			}

			@Override
			public Object getNullableResult(ResultSet rs, String columnName) throws SQLException {
				return rs.getNString(columnName);
			}

			@Override
			public Object getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
				return rs.getNString(columnIndex);
			}

			@Override
			public Object getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
				return cs.getNString(columnIndex);
            }
        }

        mybatis xml 파일에서 아래와 같이 선언해주면 끝!
        ,#{content, jdbcType=NVARCHAR,typeHandler=com.test.PreparedSatementNVarchar}  

  아래와 같이 선언시 선언된것만 정상적으로 NVARCHAR 로 저장되는것을 확인했다. 
  
