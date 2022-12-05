# defects4j-instrumented

currenly only a draft for Lang-19

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`

```java
public static final String unescapeHtml4(String input) {
  String returnValue = null;
  try {
    // Original Code START
    // return UNESCAPE_HTML4.translate(input);
    returnValue = UNESCAPE_HTML4.translate(input);
    // Original Code END
  } catch (Exception e) {
    if (input.contains("&#03")
        && e.getClass().equals(StringIndexOutOfBoundsException.class)) {
      throw new RuntimeException("Execution violates behavior specified in the bug report.");
    } else {
      throw new RuntimeException(e);
    }
  }
  return returnValue;
}
```

