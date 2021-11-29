# 前言

单元测试是一种用于测试“单元”的软件测试方法，其中“单元”的意思是指软件中各个独立的组件或模块。开发者需要为他们的代码编写测试用例以确保这些代码可以正常使用。单元测试有助于发现和防止 bugs 的出现。

在我们的业务开发中，通常应用的是敏捷开发的模型。在此类模型中，单元测试在大部分情况下是为了确保代码的正常运行以及防止在未来迭代的过程中出现问题。那么我们应该如何在业务开发中书写测试用例呢？

## 规范
### 1. 工具
    在袋鼠云数栈团队，我们建议使用 `[jest](https://github.com/facebook/jest)` + `[@testing-library/react](https://github.com/testing-library/react-testing-library)` 来书写测试用例。后者是为 `DOM` 和 `UI` 组件测试的软件工具。
### 2. 描述
    在用 `describe` 或者 `test\it` 函数时候，通常我们需要用描述当前测试用例测试的内容，我们建议**首字母大写**，并且尽量保证语句的通顺。
### 3. cleanup
    在每一个测试用例结束之后，确保所有的状态能回归到最初状态，比如在 UI 组件测试中，我们建议在 `afterEach` 中调用 `cleanup` 函数

    ```js
    import { cleanup } from '@testing-library/react';

    describe('For test', () => {
        afterEach(cleanup);
    
        test('...', () => {})
    })
    ```

### 4. 快照测试
    通常在测试 UI 组件时，我们会建议进行快照测试，以确保 UI 不会有意外的改变。这里我们建议使用 `react-test-renderer` 进行快照测试。

    ```shell
    yarn add react-test-renderer @types/react-test-renderer -D
    ```

安装完成后，建议在 UI 测试的首个测试用例进行快照测试。

    ```js
    import React from 'react';
    import renderer from 'react-test-renderer';
    import { Toolbar } from '..';

    test('Match Snapshot', () => {
    const component = renderer.create(<Toolbar data={toolbarData} />);
    const toolbar = component.toJSON();
        expect(toolbar).toMatchSnapshot();
    });
    ```

### 5. 函数命名
    关于是使用 `test` 还是使用 `it` 的争论，我们不做限制。但是建议一个项目里，尽量保持风格一致，如果其余测试用例中均为 `test`，则建议保持统一。
### 6. 业务代码
    我们建议尽量把业务代码的函数的功能单一化，简单化。如果一个函数的功能包含了十几个功能数十个功能，那我们建议对该函数进行拆分，从而更加有利于测试的进行
### 7. 重构代码
    在重构代码之前，请确保该模块的测试用例已经补全，否则重构代码的风险会过于巨大，从而导致无法控制开发成本。
### 8. 覆盖率
    我们建议尽量以覆盖率 **100% **为目标。当然，在具体的开发过程中会有各种各样的情况，所以很少有能够达到 100% 的情况出现。
### 9. 修复问题
    每当我们修复了一个 bug，我们应当评估是否有必要为这个 bug 添加一个测试用例。如果需要的话，则在测试用例中新增一条以确保后续的开发中不会复现该 bug。
    评估的参考内容如下：

    - 是否会造成白屏或其他严重的问题
    - 是否会影响用户的交互行为
    - 是否会影响内容的展示

以上内容，满足一条或多条，则认为应当为该 bug 新增测试用例。
### 10. toBe or toEqual
    这两者的区别在于，`toBe` 是相等，即 `===`，而 `toEqual` 是内容相同，即深度相等。我们建议基础类型用 `toBe`，复杂类型用 `toEqual`。

### 11. Drag
    有时候，我们需要去测试拖拽功能，我们建议用以下函数来执行模拟拖拽的操作

    ```javascript
    import { fireEvent } from '@testing-library/react';

    function dragToTargetNode(source: HTMLElement, target: HTMLElement) {
        fireEvent.dragStart(source);
        fireEvent.dragOver(target);
        fireEvent.drop(target);
        fireEvent.dragEnd(source);
    }
    ```
### 12. UI 测试
    除了快照测试外，UI 测试还需要测试 UI 组件是否如期望般渲染，那么我们建议使用 `[@testing-library/jest-dom](https://github.com/testing-library/jest-dom)` 做相关的测试

    ```shell
    yarn add --dev @testing-library/jest-dom
    ```

测试例子如下：

    ```js
    import React from 'react';
    import { render, waitFor } from '@testing-library/react';
    import '@testing-library/jest-dom';

    describe('Test Breadcrumb Component', () => {
        test('Should support to render custom title', async () => {
            const { container, getByTitle } = render(
                <MyComponent
                    renderTitle={() => "I'm renderTitle";}
                />
            );

            const testDom = await waitFor(() =>
                container.querySelector('[title="test1"]')
            );
            const dom = await waitFor(() =>
                container.querySelector('[title="I\'m renderTitle"]')
            );

            expect(testDom).not.toBeInTheDocument();
            expect(dom).toBeInTheDocument();
        });
    });
    ```

除了 `toBeInTheDocument` 外，还有其余接口，参见官方文档。
### 13. `test.only`
    在出现测试用例无法通过，但是又判断代码的**逻辑**没有问题之后，将该条测试用例设置为 `only` 再跑一遍测试用例，以确保不是其他测试用例导致的该测试用例的失败。这类问题经常出现自代码中欠缺深拷贝，导致多条测试用例之中修改了原数据从而使得数据不匹配。
    例如：
    
    ```js
    // mycode.ts
    function add(record: Record<string, any>){
        Object.assign(record, { flag: false});
    }

    // mycode.test.ts
    const mockData = {};
    test('',() => {
    add(mockData)
        // ...
        // ...
    })

    test.only('',() => {
    add(mockData) // the mockData is modified by add function here
        // ...
        // ...
    })
    ```