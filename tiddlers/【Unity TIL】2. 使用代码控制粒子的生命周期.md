一些情况下可能需要手动地结束例子的生命周期，比如在一定的时间消失，或者移动到某个地点后消失。

## 如何控制
通过控制修改粒子的 `remainingLifetime` 可以控制粒子的存活时间，为 0 时，粒子会销毁。

## 示例
```csharp
ParticleSystem ps;
//...
int maxCount = ps.main.maxParticles;
ParticleSystem.Particle[] particles = new ParticleSystem.Particle[maxCount];
//...

// 获取粒子
int count = ps.GetParticles(particles);
for(int i=0; i<count; i++)
{
    // 在达到某种需要的状态后，将 remainingLifetime 设置为 0；
    // particle 的 remainingLifetime 为 0 就会销毁
	particles[i].remainingLifetime = 0;
}
// 完成修改后，重新赋值回去
ps.SetParticles(particles);
```