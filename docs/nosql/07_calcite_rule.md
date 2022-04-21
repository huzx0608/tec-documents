# How to write Rule

## 1.RBO规则

### 1.1 RBO规则的写法

在Calcite中，创建一个Rule只需要以下步骤:

1. 确定改Rule所需要作用的RelNode及其输入 
2. 继承RelOptRule或者ConverterRule类 
3. 实现对应的onMatch()方法或convert方法

### 1.2 规则说明

```java
public class ProjectMergeRule extends RelOptRule {
  public static final ProjectMergeRule INSTANCE =
      new ProjectMergeRule(true, RelFactories.LOGICAL_BUILDER);

  /**
   * Creates a ProjectMergeRule, specifying whether to always merge projects.
   *
   * @param force Whether to always merge projects
   */
  public ProjectMergeRule(boolean force, RelBuilderFactory relBuilderFactory) {
    super(
        // 第一个RelNode的类型，
        operand(Project.class,
       // 第二个RelNode类型, 第三个RelNode类型为any(), 任意
            operand(Project.class, any())),
        relBuilderFactory,
        "ProjectMergeRule" + (force ? ":force_mode" : ""));
    this.force = force;
  }

  public void onMatch(RelOptRuleCall call) {
    final Project topProject = call.rel(0);
    final Project bottomProject = call.rel(1);
    final RelBuilder relBuilder = call.builder();

    call.transformTo(relBuilder.build());
  }
}
```

## 2.ConvertRule

ConverterRule 主要作用是将RelNode 从一个Convention转化成另一个Convention, 关于Convention的作用，我会专门用细说。
