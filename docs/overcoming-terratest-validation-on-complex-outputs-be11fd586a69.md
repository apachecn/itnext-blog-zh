# 克服复杂输出的 Terratest 验证

> 原文：<https://itnext.io/overcoming-terratest-validation-on-complex-outputs-be11fd586a69?source=collection_archive---------3----------------------->

![](img/c8f1d1a06cf919f665c7728ac5828bed.png)

Gabs 图书馆

为 Terraform 模块开发单元测试对于验证行为和进行回归测试至关重要。

最近，我不得不在 Terraform 0.14 迁移期间对大约 15 个模块进行回归测试，这些测试是我的救命稻草。然而，我发现使用 Terratest 来验证模块的输出变得过于单调乏味，因为我们要公开完整的对象，并对各种属性、数组、映射、嵌套对象等进行验证。

Terratest 是一个 golang 库，允许您调用 terraform 模块(init/apply)，验证输出，然后拆除临时基础设施。terraform 模块提供了许多 *OutputXXXX* 函数来从模块输出中提取数据，以便您可以执行断言。总的来说，它对于只有一个值或者只有一层深度的输出非常有效。一旦您开始输出完整的对象，所有的地狱打破松散试图铸造的数据。项目所有者[也承认](https://github.com/gruntwork-io/terratest/issues/709)对库嵌套的支持有限。

沮丧之余，我做了一些研究，发现了一个 JSON 库，它可以很容易地定位属性，然后访问子结构。因为 terratest 允许以 JSON 格式输出数据…我找到匹配的了！

[Gabs](https://github.com/Jeffail/gabs) 库在寻找我想要验证的子属性、遍历地图和列表、以及大大简化和减少我必须做的转换数量方面一直是救命稻草。

作为一个例子，我有一个 terraform 模块，我在其中部署了一个 Azure 防火墙和一些预烘焙的防火墙规则，并且消费者还可以添加应用程序规则。从验证的角度来看，我想确保我嵌入的观点是经过检查的，并且在未来的变更中保持不变。

感兴趣的项目:

*   公共 IP —我允许创建多个公共 IP 并连接到防火墙(以增加吞吐量)。验证标记是否级联，确保 SKU 设置为标准，验证命名
*   防火墙—确保所有公共 IP 都连接到防火墙，确保威胁英特尔模式设置为拒绝。
*   应用程序规则—确保终端用户定义的规则得到部署，验证可选属性

为了完成上面的验证，下面是展示许多 Gabs 功能的 terratest 代码示例。

```
package testsimport (
 "regexp"
 "testing""github.com/Jeffail/gabs"
 "github.com/gruntwork-io/terratest/modules/terraform"
 "github.com/stretchr/testify/assert"
)func TestModuleDeployment(t *testing.T) {
 t.Parallel()
 terraformOptions := &terraform.Options{
  TerraformDir: "../examples/",
 }defer terraform.Destroy(t, terraformOptions)
 terraform.InitAndApply(t, terraformOptions)
 validateOutputs(t, terraformOptions)
}func validateOutputs(t *testing.T, opts *terraform.Options) {
 jsonParsed, err := gabs.ParseJSON([]byte(terraform.OutputJson(t, opts, "")))
 if err != nil {
  panic(err)
 }//Firewall Validation
 fw, _ := jsonParsed.JSONPointer("/firewall/value")
 assert.Equal(t, 4, len(fw.Search("ip_configuration").Children())) //Validate 4 IP Configurations are present
 for _, child := range fw.Search("ip_configuration").Children() {
  assert.Regexp(t, regexp.MustCompile(`configuration-\d`), child.Search("name").Data().(string))
 }assert.Equal(t, "ScPcFWL-egress_fw-azfw", fw.Search("name").Data().(string)) //FW naming convention based on input
 assert.Equal(t, "Deny", fw.Search("threat_intel_mode").Data().(string))      //Threat intelligence set to Deny
 assert.Equal(t, 2, len(fw.Search("zones").Children()))                       //Must have 2 zones in CanadaCentral//Public IP Validation
 pip, _ := jsonParsed.JSONPointer("/public_ip/value")
 assert.Equal(t, 4, len(pip.Children()))for _, child := range pip.Children() {
  assert.Equal(t, "Standard", child.Search("sku").Data().(string))                                      //Ensure the SKU is set to standard
  assert.Equal(t, "true", child.Search("tags", "test").Data().(string))                                 //Ensure tags are cascaded to PIP
  assert.Regexp(t, regexp.MustCompile(`ScPcFWL-egress_fw-pip\d`), child.Search("name").Data().(string)) //PIP naming
 }//Application Rules
 appRules, _ := jsonParsed.JSONPointer("/application_rules/value")
 assert.True(t, appRules.Exists("aks"))
 assert.True(t, appRules.Exists("packages"))
 assert.Equal(t, 2, len(appRules.Children())) //2 Rules passed in .. 2 rules created//All Passed in values should have priority greater than 200\. Right now the priorities are not being enforced in the module
 for _, child := range appRules.Children() {
  intValue, _ := child.S("priority").Data().(float64)
  assert.GreaterOrEqual(t, intValue, float64(200))
 }//Description is an optional attribute. Ensuring it is populated when passed in
 descriptionPtr, _ := jsonParsed.JSONPointer("/application_rules/value/aks/rule/0/description")
 assert.Equal(t, "Sample Description", descriptionPtr.Data().(string))packageRules, _ := jsonParsed.JSONPointer("/application_rules/value/packages/rule")
 assert.Equal(t, 2, len(packageRules.Children())) // 2 rules for this application rule (docker & ubuntu)
}
```

我希望这对于通过使用 Gabs 库用 Terratest 扩展您的单元测试是有用的。