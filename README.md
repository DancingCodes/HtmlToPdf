注意：
1.items不要设置上下margin,否者会导致pdf计算错位
2.单个items不能超过一页,尽量设置更原子的items
3.Html转Pdf的本质是Html => Canvas => Image => Pdf，如果要添加页眉或其他等，尽量使用图片，使用文字要导入字体包，页面加载会变慢