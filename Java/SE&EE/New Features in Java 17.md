New Features in Java 17
--

> 目前JAVA11的新應用，已追上JAVA8的新應用，儼然已經開始進入11的世代
> 而JAVA17，使用甚至不到族群內的5%
> 幾乎全世界，都有鄉民在喊:嘿，我還在用上古java6，甚至java5，而且公司上千人甚至SP500

> 但，式微的語言相當於工程師需求量減少，雖然也不乏有COBOL強者幹到現在
> 整體而言，要確認的是，你維護的產品會有改版或退場的一天嗎?他會在你安然退休前發生嗎?你有能力應付衝擊嗎?

> 另外上古程式通常還有一個常見特性，不對外，或需要一個中繼站台在DMZ區
> 如果你開發的產品，是API或著其他對外、雲端服務，各種效能的改善、安全性的更新對你來說就很重要了
> 相應的市場也會逐步成長
> 以JAVA來說，版本釋出是一連串的故事，就算現在不使用，這至少也會是一場與JAVA一同成長得歷程

--

JAVA11可參考

New Features in Java 11

https://github.com/AngusPrimeYang/Manual/blob/main/Java/SE%26EE/New%20Features%20in%20Java%2011.md

--

(JEP-306) Restore Always-Strict Floating-Point Semantics

	浮點數不歪樓。並不是浮點數不會再有誤差，而是統一定義(IEEE 754)輸出，讓結果有所謂逐位再現性。
	過去是為了效能、以及更為廣泛的支援，而放寬浮點數語意，然而時代過去，主流CPU都已支援。
	Java使用上不再需要加關鍵字strictfp，使用了也會提醒要拿掉。
	參見上古文書 https://jcp.org/en/jsr/detail?id=84
	較好懂的浮點數定義說明 https://ithelp.ithome.com.tw/articles/10266532

(JEP-356) Enhanced Pseudo-Random Number Generators

	增強型偽亂數產生器 RandomGenerator，可以指定不同強度的演算法產生亂數
	或直接RandomGenerator.getDefault();用預設的(目前是L32X64MixRandom，Default會隨時間變化)
	在thread中會有風險，請使用ThreadLocalRandom類，也可以在thread外先初始化SplittableGenerator給thread使用
	https://www.baeldung.com/java-17-random-number-generators

(JEP-382) New macOS Rendering Pipeline
	
	macOS的Java 2D 渲染，新增Apple Metal API，減少Apple OpenGL API(deprecated)
	
(JEP-391) macOS/AArch64 Port

	jdk增加支援平台macOS/AArch64

(JEP-398) Deprecate the Applet API for Removal

	永別了Java Applets(deprecated: 9, removal: 17)

(JEP-403) Strongly Encapsulate JDK Internals

	強化runtime核心組件的安全性與封裝
	內部 API開始限制存取，因為準備多年，且幾乎有替代方案
	https://wiki.openjdk.org/display/JDK8/Java+Dependency+Analysis+Tool#JavaDependencyAnalysisTool-ReplaceusesoftheJDK'sinternalAPIs
	執行java時增加 --add-exports 或 --add-opens 等參數，可取用已預設為未開放、且允許開放使用的組件，如java.naming
	https://docs.oracle.com/en/java/javase/16/migrate/migrating-jdk-8-later-jdk-releases.html#GUID-12F945EB-71D6-46AF-8C3D-D354FD0B1781

(JEP-406) Pattern Matching for switch (Preview)

	神奇switch，這個GA的話堪比DOTNET
		case null
		case Integer
		case Student (instanceof)
		case Student s && (s.score() > 90)
		
(JEP-407) Remove RMI Activation
	
	刪除RMI啟動機制(deprecated: 15, removal: 17)

(JEP-409) Sealed Classes

	我稱之為限定繼承
	sealed、permits
	https://klab.tw/2023/01/java-17-sealed-non-sealed-and-final-class/

(JEP-410) Remove the Experimental AOT and JIT Compiler
	
	introduced: 9, removal: 17
	
(JEP-411) Deprecate the Security Manager for Removal
	
	java.lang.SecurityManager deprecated
	
--
		
(JEP-412) Foreign Function & Memory API (Incubator)
	
	目標孵化出更好用的、跨語言存取lib或應用程式的Java Native Interface ( JNI )
	
(JEP-414) Vector API (Second Incubator)
	
	目標孵化出更加改善核心效能的API
	
(JEP-415) Context-Specific Deserialization Filters

	目標更加安全、優質的序列化/反序列化工具
	透過ObjectInputFilter過濾序列化/反序列化對象

--

Reference

https://howtodoinjava.com/java/new-features/

https://mkyong.com/java/what-is-new-in-java-17/

https://www.oracle.com/tw/news/announcement/oracle-releases-java-17-2021-09-14/

https://www.baeldung.com/java-17-random-number-generators
