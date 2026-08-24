손실 함수와 목표 함수 흐름
모델 예측 -> 정답 -> 손실함수 -> 스칼라 -> Optimizer는 loss를 줄이는 방향을 paramater를 바꿉니다.

2.손실 함수는 왜 필요한가요?

2-1. 정답/오답만으로는 왜 부족한가요?

2-2. loss function과 objective function은 같은 말인가요?

3. prediction, target, loss의 관계

import torch

preds = torch.tensor()

4. MSELoss를 직접 계산해보기

1. 예측값 - 정답값
2. 차이를 제곱
3. 전체 평균

5.reduction 옵션 이해하기

#연습문제

import torch
import torch.nn as nn

preds = torch.tensor([[1.0], [2.0], [3.0]])
target = torch.tensor([[1.5], [2.5], [2.0]])

criterion = nn.MSELoss()

manual_loss = ((preds - target)**2).mean()
torch_loss = criterion(preds, target)

print("manual_loss:", manual_loss)
print("torch_loss:", torch_loss)
print("same?", torch.allclose(manual_loss, torch_loss))

A. loss = nn.MSELoss().(preds, target) # 
B. loss = torch.BCEWithLoss().(logits, target) # 0 or 1
C. loss = torch.CrossEntropyLoss().(logits, target) # dtype=torch.long

2. Optimizer는 무엇을  하나요?

step 시작
-> 이전 gradient 제거
-> 현재 parameter로 prediction 생성
-> prediction과 target으로 loss 계산
-> loss에서 parameter까지 gradient 계산
-> gradient로 parameter 업데이트
-> 다음 step

5-4강. 파라미터 업데이트 코드 흐름

학습목표
1. zero_grad, forward, loss, backward, step의 순서를 설명할 수 있습니다.
2. gradient가 누적되지 이유와 zero_grad()가 필요한 이유를 설명할 수 있습니다.
3. 학습 전후 parameter값을 비교해 업데이트가 일어났는지 확인할 수 있습니다.
4. 깨진 학습 step 코드를 보고 잘못된 순서를 수정할 수 있습니다.

1. 이번에 우리가 배울 것

optimizer.zero_grad()
outputs = model(inputs)
loss = criterion(outputs, targets)
loss.backward()
optimizer.step()

2. 학습 step의 전체 순서
한 step은 현재 paraneter로 예측하고, 오차를 계산하고, 수정 방향을 구한 뒤, 실제 값을 바꾸는 하나의 사이클입니다.

2-1. 시간 순서로 읽어봅니다
step 시작
-> 이전 gradient 제거
-> 현재 parameter로 ptediction 생성
-> prediction과 target으로 loss 계산
-> loss에서 parameter까지 gradient 계산
-> gradient로 parameter 업데이트
-> 다음 step

optimizer.step()
new_outputs = model(inputs)
new_loss = criterion(new_outputs, targerts)

optimizer.zero_grad()
loss1.backward()
grad_after_first = model.weight.grad.detach().clone()

# zero_grad 없이 다시 backward
loss2.backward()
after_second = mpdel.weight.grad.detach().clone()

print("first grad:", grad_after_first)
print("accumulate grad:", grad_after_second)

optimizer.zero_grad()
for micro_step, batch in enmerate(micro_batches):
  loss = compute_loss(batch) /accumulation_steps
  loss.backward()
optimizer.step()

import torch
import torch.nn as nn

model = nn.Linear(3, 1)
criterion = nn.MSELoss()

inputs = torch.randn(4, 3)
targets = torch.randn(4, 1)

outputs = model(inputs)
loss = criterion(outputs, targets)

print("outputs shape:", outputs.shape)
print("targets shape:", targets.shape)
print("loss:", loss)

print(model.weight.grad) # 보통
loss.backward()
print(model.weight.grad) # Tensor 생성

expected = weight_before -lr * weight_grad
optimizer.step()
actual = model.weight.detach().clone()
print(torch.allclose(expected, actual))

with torch.no_grad():
  for param in model.parameters():
    param -= lr *param.grad

weight_before = model.weight.detach().clone()

7-2. 검증 순서

1. weight_before 복사
2. zero_grad
3. forward와 loss
4. backward 후 weight_grad 복사
5. expected-weight 계산
6. step
7. weight_after와 expected_weight 비교

# 7-3. parameter가 정말 바뀌었는지  세 가지로 확인합니다.

changed = not torch.equal(weight_before, weight_after)
close_to_expected = torch.allclose(weight_after, experted_weight)
updatr_norm = (weight_after - weight_before).norm().item()

print("changed:", changed)
print("matches SGD formula:", close_to_expected)
print("update norm:", update_norm)

import torch
import torch.nn as nn
import torch.optim as optim

# 재현을 위해 seed를 고정합니다.
torch.manual_seed(0)

model = nn.Linear(3, 1)
criterrion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.1)

inputs = torch.randn(4, 3)
targets = torch.randn(4, 1)

# 업데이트 전 parameter를 복사해둡니다.
weight_before = model.weight.detach().clone()
bias_before = model.bias.detach().clone()

# 1. gradient 초기화
optimizer.zero_grad()

# 2. forward
outputs = model(inputs)

# 3. loss 계산
loss = criterion(outputs, targets)

# 4. backward
gradient_before_backward =model.weight.grad
loss.backward()

# step 직전에 계산된 gradient를 복사합니다.
weight_grad = model.weight.grad.detach().clone()
bias_grad = model.bias.grad.detach().clone()

# momentum이 없는 SGD의 예상 업데이트를 직접 계산합니다.
expected_weight = weight_before - 0.1 * weight_grad
expected_bias = bias_before - 0.1 * bias_grad

# 5. step
optimizer.step()

weight_after = model.weight.detach().clone()
bias_after = model.bias.detach().clone()

print("gradient before backward:", gradient_before_backward)
print("weight before:", weight_before)
print("weight grad  :", weight_grad)
print("weight expected after:", expected_weight)
print("weight actual after  :", weight_after)
print("bias before:", bias_before)
print("bias grad  :", bias_grad)
print("bias expected after:", expected_bias)
print("bias actual after  :", bias_after)
print("weight formula matches?", torch.allclose(expected_weight, weight_after))
print("bias formula matches?  ", torch.allclose(expected_bias, bias_after))

import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(5, 2)
criterion = nn.CrossEntropyLoss()

x = torch.randn(8, 5)
logits = model(x)
target = torch.tensor([[0], [1], [0], [0], [1], [0], [1], [1]], index=class(2), dtype=torch.long)

optimizer =optim.Adam(model.parameters(), lr=0.001)

optimizer.zero_grad = model.torch.tensor.gradient()
loss = criterion(logits, target)
loss.backward

after_grad = model.torch.tensor.gradient()

optimizer.step()

import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(5, 2)
criterion = nn.CrossEntropyLoss()

x = torch.randn(8, 5)
logits = model(x)
target = torch.tensor([[0], [1], [0], [0], [1], [0], [1], [1]], index=class(2), dtype=torch.long)

optimizer =optim.Adam(model.parameters(), lr=0.001)

optimizer.zero_grad = model.torch.tensor.gradient()
loss = criterion(logits, target)
loss.backward

after_grad = model.torch.tensor.gradient()

optimizer.step()

import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(5, 2)
criterion = nn.CrossEntropyLoss()

x = torch.randn(8, 5)
logits = model(x)
target = torch.tensor([0, 1, 0, 0, 1, 0, 1, 1], dtype=torch.long)

weight_before = model.weight.detach().clone()

optimizer =optim.Adam(model.parameters(), lr=0.001)

optimizer.zero_grad()
loss = criterion(logits, target)

gradient_before_backward = model.weight.grad
loss.backward()

weight_grad = model.weight.grad.detach().clone()

optimizer.step()