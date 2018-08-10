This is a minimal testcase that demonstrates what I believe is a bug in Git 2.18 and 2.14.

# Overview #

When interactively selecting hunks to apply, using `git checkout -p <tree> <file>`, git will correctly apply an unmodified hunk, but will refuse to apply a hunk that went through a text editor ("e" command), even when I made no changes in the text editor.

# Minimal test case #

This repository has two branches:

* `numbers-1-3`, having a file with three numbered lines.
* `just-2`, having a file with just one numbered line, "2".

Steps to reproduce:

1. `git checkout just-2`: Check out the `just-2` branch.

2. `git checkout -p numbers-1-3 numbers.txt`: Interactively patch `numbers.txt` with additional lines.  We care only about the second hunk ("3").

```
% git checkout -p numbers-1-3 numbers.txt
diff --git b/numbers.txt a/numbers.txt
index 0cfbf08..01e79c3 100644
--- b/numbers.txt
+++ a/numbers.txt
@@ -1 +1,3 @@
+1
 2
+3
Apply this hunk to index and worktree [y,n,q,a,d,s,e,?]? 
```

3. Split this hunk.

```
Apply this hunk to index and worktree [y,n,q,a,d,s,e,?]? s
Split into 2 hunks.
@@ -1 +1,2 @@
+1
 2
Apply this hunk to index and worktree [y,n,q,a,d,j,J,g,/,e,?]?
```

4. It does not matter whether you take this hunk or not.  In this example, I Do *n*ot take the first sub-hunk ("1").

```
Apply this hunk to index and worktree [y,n,q,a,d,j,J,g,/,e,?]? n
@@ -1 +2,2 @@
 2
+3
Apply this hunk to index and worktree [y,n,q,a,d,K,g,/,e,?]?
```

5. When offered to take the second sub-hunk ("3"), choose to *e*dit.  Your text editor will open.

```
Apply this hunk to index and worktree [y,n,q,a,d,K,g,/,e,?]? e
```

6. Exit the text editor without making any changes.

*Expected behavior:* git applies the hunk.

*Actual behavior:*

```
error: patch failed: numbers.txt:1
error: numbers.txt: patch does not apply
Your edited hunk does not apply. Edit again (saying "no" discards!) [y/n]?
```

# Variations that behave as expected #

1. Accepting the second hunk without going through the text editor
2. Accepting the full original hunk without splitting it, regardless of whether you go through a text editor or not.
