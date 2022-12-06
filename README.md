# defects4j-instrumented

This repository includes the instrumented defects4j subjects. We manually transformed the information from the bug report into an oracle, which is added to the buggy version of each subject.

Each modification is tagged as: `D4J_<project>_<bugID>_BUGGY_VERSION_INSTRUMENTED`

The zipped git archives are included in [instrumented-archives](./instrumented-archives).

*Note: For now, we will only have direct modifications of the buggy source code. Later, we want to introduce an automated mechanism that (given the source code modification) directly instruments the Java bytecode.*

## Current general issues:

* not all test cases call the method mentioned in the bug report; they call the underlying buggy method

## Example: Lang-19

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`

```diff
yannic@yannics-mbp lang_19 % git diff D4J_Lang_19_BUGGY_VERSION D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED
diff --git a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
index d84aae58..2a0d1f3d 100644
--- a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
+++ b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
@@ -465,7 +465,17 @@ public class StringEscapeUtils {
      * @since 3.0
      */
     public static final String unescapeHtml4(String input) {
-        return UNESCAPE_HTML4.translate(input);
+        try {
+            // Original Code START
+            return UNESCAPE_HTML4.translate(input);
+            // Original Code END
+        } catch (StringIndexOutOfBoundsException e) {
+            if (input.contains("&")) {
+                throw new RuntimeException("Execution violates behavior specified in the bug report.");
+            } else {
+                throw e;
+            }
+        }
     }
```

## Progress

The following list includes the already covered subjects.

<details>
<summary><b>Lang-19</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`

```java
public static final String unescapeHtml4(String input) {
    try {
        // Original Code START
        return UNESCAPE_HTML4.translate(input);
        // Original Code END
    } catch (StringIndexOutOfBoundsException e) {
        if (input.contains("&")) {
            throw new RuntimeException("Execution violates behavior specified in the bug report.");
        } else {
            throw e;
        }
    }
}
```

</details>
