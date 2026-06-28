---
title: "10 MyBatis SQL Injection Vulnerabilities AI Can Catch That Humans Miss"
date: 2026-06-25
draft: false
tags: ["mybatis", "sql-injection", "security", "ai-code-review", "java"]
description: "Discover 10 subtle MyBatis SQL injection vulnerabilities that are easy to miss in code review. Learn how AI-powered code analysis can catch these security risks automatically."
---

MyBatis is the most popular ORM framework in the Java ecosystem, powering millions of applications. But its flexibility comes with a dangerous gotcha: **`${}` vs `#{}`** syntax. One is safe, the other is not — and the difference is a single character.

Here are 10 MyBatis SQL injection patterns that slip past human reviewers but an AI code review agent catches instantly.

## The Core Problem: `${}` vs `#{}`

```xml
<!-- SAFE: #{} uses PreparedStatement parameter binding -->
<select id="findById" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>

<!-- VULNERABLE: ${} directly interpolates the string -->
<select id="findById" resultType="User">
    SELECT * FROM users WHERE id = ${id}
</select>
```

The difference: `#{id}` generates `WHERE id = ?` with parameter binding, while `${id}` generates `WHERE id = 1 OR 1=1` with direct string interpolation.

Let's look at 10 patterns where this becomes a real vulnerability.

## 1. Dynamic Column Names

```xml
<select id="findByColumn" resultType="User">
    SELECT * FROM users 
    ORDER BY ${sortColumn} ${sortDirection}
</select>
```

This is the most common legitimate use of `${}` — dynamic column names and sort directions. But if `sortColumn` comes from user input, it's exploitable:

```
Attack: sortColumn = "name; DROP TABLE users; --"
```

**Fix:** Validate against a whitelist:

```java
private static final Set<String> ALLOWED_COLUMNS = 
    Set.of("name", "created_at", "email", "id");

public List<User> findByColumn(String sortColumn, String direction) {
    if (!ALLOWED_COLUMNS.contains(sortColumn)) {
        throw new IllegalArgumentException("Invalid sort column");
    }
    direction = "ASC".equalsIgnoreCase(direction) ? "ASC" : "DESC";
    return mapper.findByColumn(sortColumn, direction);
}
```

## 2. Dynamic Table Names

```xml
<select id="queryFromTable" resultType="Map">
    SELECT * FROM ${tableName} WHERE status = #{status}
</select>
```

User-controlled table names are extremely dangerous:

```
Attack: tableName = "users JOIN credit_cards ON users.id = credit_cards.user_id"
```

**Fix:** Use a mapping enum:

```java
public enum QueryTable {
    USERS("users"),
    ORDERS("orders"),
    PRODUCTS("products");
    
    private final String tableName;
    // constructor and getter
}
```

## 3. IN Clause Construction

```xml
<select id="findByIds" resultType="User">
    SELECT * FROM users WHERE id IN (${ids})
</select>
```

With `<foreach>` this is safe, but manual string concatenation is not:

```java
// VULNERABLE
String ids = request.getIds().stream()
    .collect(Collectors.joining(","));
mapper.findByIds(ids);  // "1,2) OR 1=1 --"
```

**Fix:** Use `<foreach>`:

```xml
<select id="findByIds" resultType="User">
    SELECT * FROM users WHERE id IN
    <foreach item="id" collection="ids" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

## 4. LIKE Clause Injection

```xml
<select id="findByName" resultType="User">
    SELECT * FROM users WHERE name LIKE '%${keyword}%'
</select>
```

Even `#{}` doesn't fully protect LIKE clauses from logic injection:

```
Input: keyword = "admin'--"
Generated: WHERE name LIKE '%admin'--%'
```

**Fix:** Escape special characters and use `CONCAT`:

```xml
<select id="findByName" resultType="User">
    SELECT * FROM users 
    WHERE name LIKE CONCAT('%', #{keyword}, '%') ESCAPE '\'
</select>
```

## 5. Annotation-Based Mappers

```java
@Select("SELECT * FROM users WHERE role = ${role}")
List<User> findByRole(String role);
```

Annotation mappers are just as vulnerable as XML mappers. The AI agent catches these by scanning `@Select`, `@Update`, `@Insert`, `@Delete` annotations.

## 6. MyBatis-Plus QueryWrapper Abuse

```java
// VULNERABLE: apply with raw SQL
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.apply("name = '" + name + "'");  // SQL injection!
```

MyBatis-Plus's `apply()` method accepts raw SQL. If you concatenate user input, it's injection-prone.

**Fix:** Use parameter binding in `apply()`:

```java
wrapper.apply("name = {0}", name);  // Safe parameter binding
```

## 7. Dynamic WHERE Conditions

```xml
<select id="dynamicQuery" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null">
            AND name = '${name}'
        </if>
        <if test="email != null">
            AND email = '${email}'
        </if>
    </where>
</select>
```

Each `${}` usage in dynamic conditions is a potential injection point.

**Fix:** Replace all `${}` with `#{}` unless you truly need literal interpolation:

```xml
<if test="name != null">
    AND name = #{name}
</if>
```

## 8. Schema Migration Scripts

```xml
<!-- In a migration XML that processes user data -->
<select id="migrateUser" resultType="Map">
    SELECT ${columns} FROM ${sourceTable} WHERE ${condition}
</select>
```

Schema migration code often gets less security review because it's "admin-only." But if any parameter traces back to user input, it's exploitable.

## 9. Stored Procedure Calls with `${}`

```xml
<select id="callProcedure" statementType="CALLABLE">
    {call ${procedureName}(#{param1}, #{param2})}
</select>
```

Dynamic procedure names are rarely needed and extremely dangerous.

## 10. Batch Operations

```xml
<update id="batchUpdate">
    <foreach collection="list" item="item" separator=";">
        UPDATE ${item.tableName} SET ${item.column} = #{item.value} 
        WHERE id = #{item.id}
    </foreach>
</update>
```

Combining `${}` with batch operations multiplies the attack surface.

## How AI Code Review Catches These

An AI code review agent can detect these patterns by scanning for:

1. **`${}` usage in mapper XML files** — Flag every occurrence
2. **User input tracing** — Follow the data from controller → service → mapper
3. **Missing whitelist validation** — Check if `${}` parameters are validated
4. **Annotation mapper scanning** — Check `@Select`/`@Update` annotations
5. **MyBatis-Plus raw SQL** — Detect `apply()` with string concatenation

Sample AI review output:

```
🚨 CRITICAL: SQL Injection vulnerability in UserMapper.xml:42

  ${sortColumn} receives user input without whitelist validation.
  
  Current code:
    ORDER BY ${sortColumn} ${sortDirection}
  
  Suggested fix:
    // In UserService.java
    private static final Set<String> SORT_COLUMNS = 
        Set.of("id", "name", "created_at");
    
    if (!SORT_COLUMNS.contains(sortColumn)) {
        throw new BusinessException("Invalid sort column");
    }
```

## Setting Up Automated Detection

Create a custom rule in your AI code review configuration:

```java
@AiService
public interface SecurityReviewAgent {
    
    @SystemMessage("""
        You are a security-focused code reviewer. For each mapper file:
        1. Flag ALL uses of ${} interpolation
        2. Trace each ${} parameter to its source
        3. Check if the parameter is validated/sanitized
        4. Rate severity: CRITICAL if user-controllable, WARNING otherwise
        5. Provide the exact fix code
        
        Report format:
        [SEVERITY] File:Line - Description
        Fix: code snippet
        """)
    String reviewMapperSecurity(@UserMessage String mapperContent);
}
```

## Conclusion

MyBatis gives you power, but `${}` gives attackers power over your database. Every `${}` usage should be treated as a potential vulnerability until proven otherwise.

The 10 patterns above cover the most common attack vectors. An AI-powered code review agent can catch these automatically, but the underlying principle is simple: **always use `#{}` unless you have a documented reason not to, and always validate `${}` parameters.**

*Want to automate this in your CI/CD pipeline? Check out our [AI Code Review Tool](https://github.com/yourusername/ai-code-review).*
