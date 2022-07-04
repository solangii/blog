---
title: "📌 PyTorch Snippets"
date: 2022-04-02T00:59:49+09:00
categories: 'snippet'
tags: ['pytorch']
draft: False
math: True
---



 Pytorch를 사용하면서 알게된 사소하지만 은근 중요한 정보(꿀팁)들을 정리합니다. 사소한 내용들이라 잘 까먹기도 해서, 틈날때 다시  기억 상기 목적으로 기록합니다. 내용은 언제든지 추가될 수 있습니다. 



## Pytorch Tensor

- Tensor와 Parameter의 차이: Tensor는 `requires_grad`의 default 설정이 **False**이나 parameter는 `requires_grad` 의 default 설정이 **True**이다.

- Tensor의 Shape을 조작해야하는 경우 메모리 저장상태가 비연속적으로 변경되는지(ex, `transpose()`, `expand()`, `view()`)  유의해야한다. 경우에 따라 `contiguous()`, `reshape()`등을 사용하여 연속적인 메모리 저장상태로 전환시켜줘야 한다.
- underbar의 이용 - `torch.add()` 와 `torch.add_()`의 차이  [^1]
  - underbar 있는 경우 (`add_()`): in_place 연산, 새로운 tensor를 만들지 않고 기존 tensor에 연산을 한다.
  - underbar 없는 경우 (`add()`): 새로운 tensor를 return



## Pytorch Autograd [^2] [^3]

- graph를 한번 만들고 재 사용하려면 `retain_graph=True` 옵션을 사용해야하고, `backward()`연산을 여러번 해줄 때에는 gradient 값이 계속 더해지기 때문에 `x.grad.zero_()`와 같은 gradient초기화 과정이 필요하다.

- autograd에서 지원하지 않는 연산을 해야할 때는 forward, backward연산을 직접 구현해야하며, 이 때 주의사항은 
  1. class를 만들 때 `torch.autograd.Function`를 상속하기
  2. forward할 때 결과를 따로 기록하기 (`ctx.save_for_backward()` 이용)
  3. backward를 직접 구현했을 때, backward가 정상적으로 작동하는지 확인할 때는 `torch.autograd.gradcheck()` 를 호출하여 확인하는 것이 유용하다.
  
- evaluation 과정 같이 backward를 하지 않는 경우에는 메모리 사용량을 줄이기 위해 `torch.no_grad()`를 사용하는 것이 좋다. 해당 부분에서 모든 변수들의 `requires_grad=False`로 지정이 된다.

- `create_graph`의 역할 : 만약 2차 미분이 필요할 경우, 내부 그래프에서 연산 식을 따로 저장을 안하기 때문에 2차 미분을 구하려하면 에러가 난다. 이때, `create_graph=True`로 해주면 그래프 내부에서 거쳐가는 노드들의 편미분 연산식들을 모두 저장하기 때문에 가능해진다. (메모리 cost는 많이 커질 것 같다)

- 중간에 gradient 값을 임의로 변경해야할 때, 변경한 `grad`값은 이후 계산에도 영향을 미친다. 

  

## Pytorch Module

Module은 주로 pytorch에 내장되어있는 여러 Module들을 조합해서 새로운 Module을 만들 때 직접 Module을 정의해서 사용하게 된다.

- `class my_module(torch.nn.Module)`을 정의하면 `self.__init__()`에선 다른 모듈들을 불러오는데 이때 `Module.__setattr__()` 내부에서 `add_module()`이 작용해서 포함한 모듈을 잘 인식할 수 있게 된다.
  - 이때 `__init__`에서 사용할 수 있는 정의 방법 중 꿀팁이라고 생각되는 게 `torch.nn.ModuleList()`, `torch.nn.ModuleDict()` 방법이다. 각각 list형태로 혹은 dict형태로 모듈을 저장함과 동시에 자동으로 모듈을 인식할 수 있게끔 내장 함수에서 처리해주기 때문에 편리한 듯 하다. 

- Module 파트에서는 Module을 여러개 묶어 사용할 때, 혹은 모듈 안에서 다른 모듈을 호출 할 때, `add_module()` 함수가 (자동이든 수동이든) 호출되어 모듈을 인식할 수 있게끔 하는 것이 중요 point였다.
  - `add_module()`을 수동으로 호출
  - 파이썬 리스트 대신 `ModuleList()`사용, 파이썬 딕셔너리 대신 `ModuleDict()`사용



## Pytorch Parameter

- Parmeter를 모듈에 추가할 때 `register_parameter()` 함수가 (자동/수동으로) 호출되어 모듈의 파라미터로 인식할 수 있게끔 하는 것이 중요하다.
  - `register_parameter()`를 수동으로 호출
  - 파이썬 리스트, 딕셔너리 대신 `ParameterList()`, `ParameterDict()` 사용



## Pytorch Buffer

- Buffer는 Module에 저장해두고 학습에 사용하진 않는 Tensor. `register_buffer()` 를 이용해 정의한다. <u>Buffer는 train mode일 때만 업데이트가 되고 eval mode에서는 값이 변하지 않는다.</u> 
  
  -  train mode일때만 업뎃 + eval mode에선 변하지 않는 변수 중 가장 먼저 떠오르는 건 역시 BN이다. Batch Normalization의 moving average mean, moving average variance 값을 저장할 때 buffer 사용한다.
  
  

## Pytorch Hook 

- Hook은 Module 내부를 건들이지 않으면서 Module의 forward, backward의 값들을 관찰/수정할 때 사용한다.





[^1]: https://stackoverflow.com/questions/52920098/what-does-the-underscore-suffix-in-pytorch-functions-mean
[^2]:https://pytorch.org/docs/stable/notes/autograd.html 
[^3]:https://teamdable.github.io/techblog/PyTorch-Autograd