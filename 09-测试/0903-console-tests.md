# 控制台测试

## 简介

除了简化 HTTP 测试之外，Laravel 还提供了一个简单的 API，用于测试需要用户输入的控制台应用程序。

## 期望输入 / 输出

Laravel 允许你使用 `expectsQuestion` 方法为你的控制台命令轻松『模仿』用户输入。另外，你可以使用 `assertExitCode` 和 `expectsOutput` 方法指定通过控制台命令你预期输出的退出代码和文本。例如：考虑如下控制台命令：

```php
Artisan::command('question', function () {
    $name = $this->ask('What is your name?');

    $language = $this->choice('Which language do you program in?', [
        'PHP',
        'Ruby',
        'Python',
    ]);

    $this->line('Your name is '.$name.' and you program in '.$language.'.');
});
```

***

你可以使用下面的测试来测试这个命令，该测试利用 `expectsQuestion`、`expectsOutput` 和 `assertExitCode` 方法：

```php
/**
 * 测试一个控制台命令。
 *
 * @return void
 */
public function test_console_command()
{
    $this->artisan('question')
         ->expectsQuestion('What is your name?', 'Taylor Otwell')
         ->expectsQuestion('Which language do you program in?', 'PHP')
         ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
         ->assertExitCode(0);
}
```

***
