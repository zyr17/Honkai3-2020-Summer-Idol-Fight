# Honkai 3 2020 Summer idol fight simulate

摸鱼期间写了个模拟来指导（玄学）下注。前段为定义，中段为模拟过程展示，尾段为大量模拟后结果


```python
import random
```


```python
class Player:
    def __init__(self, name, attack, defend, speed):
        self.name = name
        self.attack = attack
        self.defend = defend
        self.speed = speed
        self.t_attack = 0
        self.t_defend = 0
        self.t_speed = 0
        self.hp = 100
        self.atk_mul = 1
        self.hitrate = 1
        self.down = 0
        self.mute = 0
        self.counter = 0
        self.show_log = False
        self.is_element = False
        self.use_skill_skip_attack = False
    
    def do_attack(self, enemy):
        self.counter += 1
        if self.mute == 0:
            self._pre_attack(enemy)
        if self.use_skill_skip_attack:
            pass
        elif self.down > 0:
            self.down -= 1
            self.logger('down, do nothing')
        elif random.random() >= self.hitrate:
            self.logger('missed!')
        else:
            self.logger('attack enemy %d' % self.get_attack())
            enemy.damage(self.get_attack(), self.is_element, self.atk_mul)
        self._after_attack(enemy)
    
    def damage(self, atk, element = False, mul = 1):
        if element:
            atk *= mul
            self.hp -= atk
            self.logger('get %d element damage, lost %d hp, remain %d' % (atk, atk, self.hp))
        else:
            d = self.get_defend()
            dmg = atk - d if atk > d else 0
            dmg *= mul
            self.hp -= dmg
            self.logger('get %d damage, lost %d hp, remain %d' % (atk, dmg, self.hp))
    
    def _pre_attack(self, enemy):
        pass
    
    def _after_attack(self, enemy):
        self.t_attack = 0
        self.is_element = False
        self.use_skill_skip_attack = False
        if self.down > 0:
            self.down -= 1
        if self.mute > 0:
            self.mute -= 1
    
    def get_speed(self):
        return self.speed + self.t_speed
    
    def get_defend(self):
        return self.defend + self.t_defend
    
    def get_attack(self):
        return self.attack + self.t_attack
    
    def logger(self, *msg):
        if self.show_log:
            print(self.name, *msg)
    
class Kiana(Player):
    def __init__(self):
        super().__init__('Kiana', 24, 11, 23)
        
    def _pre_attack(self, enemy):
        if self.counter % 2 == 0 and self.counter > 0:
            self.t_attack = enemy.defend * 2
            self.logger('attack up by %d' % self.t_attack)
    
    def _after_attack(self, enemy):
        if self.counter % 2 == 0 and self.counter > 0:
            if random.random() < 0.35:
                self.logger('make herself down')
                self.down = 2
        super()._after_attack(enemy)
                
class Mei(Player):
    def __init__(self):
        super().__init__('Mei', 22, 12, 30)
        
    def _pre_attack(self, enemy):
        if self.counter % 2 == 0:
            self.use_skill_skip_attack = True
            self.logger('use dragon blaze!')
            enemy.damage(15, True)
        
    def _after_attack(self, enemy):
        if not self.mute and random.random() < 0.3:
            self.logger('make enemy down')
            enemy.down = 1
        super()._after_attack(enemy)

class Bronya(Player):
    def __init__(self):
        super().__init__('Bronya', 21, 10, 20)
        
    def _pre_attack(self, enemy):
        if self.counter % 3 == 0:
            self.use_skill_skip_attack = True
            self.logger('motor bike da!')
            enemy.damage(random.randint(1, 100), True)
        
    def _after_attack(self, enemy):
        if not self.mute and random.random() < 0.25:
            self.logger('use drill!')
            for i in range(4):
                enemy.damage(12)
        super()._after_attack(enemy)
                
class Himeko(Player):
    def __init__(self):
        super().__init__('Himeko', 23, 9, 12)
        
    def _pre_attack(self, enemy):
        if not self.mute and self.counter % 2 == 0:
            self.hitrate -= 0.35
            if self.hitrate < 0:
                self.hitrate = 0
            self.attack *= 2
        double = enemy.name == 'SakuraKallen' or enemy.name == 'Olenyeva' or enemy.name == 'Durandal'
        if double:
            self.atk_mul = 2
        else:
            self.atk_mul = 1
                
class Rita(Player):
    def __init__(self):
        super().__init__('Rita', 26, 11, 17)
        self.enemy_weak = False
    
    def _pre_attack(self, enemy):
        if random.random() < 0.35:
            self.logger('maid care')
            self.t_attack -= 3
            enemy.attack -= 4
            if enemy.attack < 0:
                enemy.attack = 0
        if self.counter % 4 == 0:
            enemy.atk_mul = 0.6
            enemy.hp += 4
            enemy.mute += 2
                
class SakuraKallen(Player):
    def __init__(self):
        super().__init__('SakuraKallen', 20, 9, 18)
        
    def _pre_attack(self, enemy):
        if random.random() < 0.3:
            self.hp += 25
            if self.hp > 100:
                self.hp = 100
            self.logger('eat onigiri, hp to %d' % self.hp)
        if self.counter % 2 == 0:
            self.use_skill_skip_attack = True
            self.logger('BIG onigiri')
            enemy.damage(25, True)
                
class Raven(Player):
    def __init__(self):
        super().__init__('Raven', 23, 14, 14)
        
    def _pre_attack(self, enemy):
        if enemy.name == 'Kiana' or random.random() < 0.25:
            self.logger('not only hurts you')
            self.atk_mul = 1.25
        if self.counter % 3 == 0 and self.counter > 0:
            self.use_skill_skip_attack = True
            self.logger('MY ISLAND')
            for i in range(7):
                enemy.damage(16, mul = self.atk_mul)
                
class Theresa(Player): # TODO will hit double damage by Himeko?
    def __init__(self):
        super().__init__('Theresa', 19, 12, 22)
        
    def _pre_attack(self, enemy):
        if self.counter % 3 == 0:
            self.use_skill_skip_attack = True
            self.logger('online kick')
            for i in range(5):
                enemy.damage(16, mul = self.atk_mul)
            
    def _after_attack(self, enemy):
        if random.random() < 0.3:
            self.logger('blood judas is the most cute')
            enemy.defend -= 5
            if enemy.defend < 0:
                enemy.defend = 0
        super()._after_attack(enemy)
                
class Olenyeva(Player):
    def __init__(self):
        super().__init__('Olenyeva', 18, 10, 10)
        self.revive = True
        self.star = False
        
    def _pre_attack(self, enemy):
        if self.star:
            self.use_skill_skip_attack = True
            self.logger('become star!')
            if random.random() < 0.5:
                enemy.damage(233)
            else:
                enemy.damage(50)
            self.star = False
    
    def damage(self, *argv):
        super().damage(*argv)
        if self.hp <= 0 and self.revive:
            self.logger('96C live water')
            self.hp = 20
            self.revive = False
            self.star = True
                
class Seele(Player):
    def __init__(self):
        super().__init__('Seele', 23, 13, 26)
        self.is_black = False
        
    def _pre_attack(self, enemy):
        self.logger('model change!')
        if self.is_black:
            self.t_attack = 10
            self.t_defend = -5
            self.hp += random.randint(1, 15)
        else:
            self.t_attack = -5
            self.t_defend = 10
        self.is_black = not self.is_black
                
class Durandal(Player):
    def __init__(self):
        super().__init__('Durandal', 19, 10, 15)
        
    def _pre_attack(self, enemy):
        self.logger('mo yu')
        self.attack += 3
        
    def damage(self, *argv):
        # TODO: only bounce skill
        if self.enemy.use_skill_skip_attack and random.random() < 0.16:
            self.logger('bounce')
            self.enemy.damage(30)
        else:
            super().damage(*argv)
                
class Fuhua(Player):
    def __init__(self):
        super().__init__('Fuhua', 17, 15, 16)
        
    def _pre_attack(self, enemy):
        self.is_element = True
        if self.counter % 3 == 0:
            self.use_skill_skip_attack = True
            self.logger('ink')
            enemy.damage(18, True)
            enemy.hitrate -= 0.25
```


```python
class Fighter:
    def __init__(self, a, b, showlog):
        self.a = a
        self.b = b
        self.showlog = showlog
        
    def fight(self):
        a = self.a()
        b = self.b()
        a.enemy = b
        b.enemy = a
        if self.showlog:
            a.show_log = b.show_log = True
        count = 0
        while a.hp > 0 and b.hp > 0:
            count += 1
            if self.showlog:
                print(' - round %d - ' % count)
            if a.get_speed() > b.get_speed():
                order = [a, b]
            elif a.get_speed() < b.get_speed():
                order = [b, a]
            else:
                if random.random() < 0.5:
                    order = [a, b]
                else:
                    order = [b, a]
            for i in range(2):
                order[i].do_attack(order[1 - i])
                if b.hp <= 0:
                    if self.showlog:
                        print(a.name, 'win\n----------')
                    return True
                if a.hp <= 0:
                    if self.showlog:
                        print(b.name, 'win\n----------')
                    return False
```


```python
def winrate(A, B, showlog = False, times = 100000):
    f = Fighter(A, B, showlog)
    count = 0
    for i in range(times):
        if f.fight():
            count += 1
    print(count / times)
```

# Simulates


```python
winrate(Kiana, Mei, True, 2)
```

     - round 1 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 89
    Kiana attack enemy 24
    Mei get 24 damage, lost 12 hp, remain 88
     - round 2 - 
    Mei use dragon blaze!
    Kiana get 15 element damage, lost 15 hp, remain 74
    Kiana attack up by 24
    Kiana attack enemy 48
    Mei get 48 damage, lost 36 hp, remain 52
     - round 3 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 63
    Kiana attack enemy 24
    Mei get 24 damage, lost 12 hp, remain 40
     - round 4 - 
    Mei use dragon blaze!
    Kiana get 15 element damage, lost 15 hp, remain 48
    Kiana attack up by 24
    Kiana attack enemy 48
    Mei get 48 damage, lost 36 hp, remain 4
     - round 5 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 37
    Kiana attack enemy 24
    Mei get 24 damage, lost 12 hp, remain -8
    Kiana win
    ----------
     - round 1 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 89
    Kiana attack enemy 24
    Mei get 24 damage, lost 12 hp, remain 88
     - round 2 - 
    Mei use dragon blaze!
    Kiana get 15 element damage, lost 15 hp, remain 74
    Kiana attack up by 24
    Kiana attack enemy 48
    Mei get 48 damage, lost 36 hp, remain 52
     - round 3 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 63
    Kiana attack enemy 24
    Mei get 24 damage, lost 12 hp, remain 40
     - round 4 - 
    Mei use dragon blaze!
    Kiana get 15 element damage, lost 15 hp, remain 48
    Kiana attack up by 24
    Kiana attack enemy 48
    Mei get 48 damage, lost 36 hp, remain 4
    Kiana make herself down
     - round 5 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 37
    Kiana down, do nothing
     - round 6 - 
    Mei use dragon blaze!
    Kiana get 15 element damage, lost 15 hp, remain 22
    Mei make enemy down
    Kiana attack up by 24
    Kiana down, do nothing
     - round 7 - 
    Mei attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 11
    Kiana attack enemy 24
    Mei get 24 damage, lost 12 hp, remain -8
    Kiana win
    ----------
    1.0



```python
winrate(SakuraKallen, Seele, True, 2)
```

     - round 1 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 91
    SakuraKallen eat onigiri, hp to 100
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 100
     - round 2 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 76
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 88
     - round 3 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 67
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 88
     - round 4 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 43
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 71
     - round 5 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 34
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 71
     - round 6 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 10
    SakuraKallen eat onigiri, hp to 35
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 48
     - round 7 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 26
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 48
     - round 8 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 2
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 29
     - round 9 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain -7
    Seele win
    ----------
     - round 1 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 91
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 100
     - round 2 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 67
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 80
     - round 3 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 58
    SakuraKallen eat onigiri, hp to 83
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 80
     - round 4 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 59
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 57
     - round 5 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 50
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 57
     - round 6 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain 26
    SakuraKallen BIG onigiri
    Seele get 25 element damage, lost 25 hp, remain 34
     - round 7 - 
    Seele model change!
    Seele attack enemy 18
    SakuraKallen get 18 damage, lost 9 hp, remain 17
    SakuraKallen attack enemy 20
    Seele get 20 damage, lost 0 hp, remain 34
     - round 8 - 
    Seele model change!
    Seele attack enemy 33
    SakuraKallen get 33 damage, lost 24 hp, remain -7
    Seele win
    ----------
    0.0



```python
winrate(Bronya, SakuraKallen, True, 2)
```

     - round 1 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 88
    Bronya use drill!
    SakuraKallen get 12 damage, lost 3 hp, remain 85
    SakuraKallen get 12 damage, lost 3 hp, remain 82
    SakuraKallen get 12 damage, lost 3 hp, remain 79
    SakuraKallen get 12 damage, lost 3 hp, remain 76
    SakuraKallen eat onigiri, hp to 100
    SakuraKallen attack enemy 20
    Bronya get 20 damage, lost 10 hp, remain 90
     - round 2 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 88
    Bronya use drill!
    SakuraKallen get 12 damage, lost 3 hp, remain 85
    SakuraKallen get 12 damage, lost 3 hp, remain 82
    SakuraKallen get 12 damage, lost 3 hp, remain 79
    SakuraKallen get 12 damage, lost 3 hp, remain 76
    SakuraKallen BIG onigiri
    Bronya get 25 element damage, lost 25 hp, remain 65
     - round 3 - 
    Bronya motor bike da!
    SakuraKallen get 28 element damage, lost 28 hp, remain 48
    SakuraKallen attack enemy 20
    Bronya get 20 damage, lost 10 hp, remain 55
     - round 4 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 36
    Bronya use drill!
    SakuraKallen get 12 damage, lost 3 hp, remain 33
    SakuraKallen get 12 damage, lost 3 hp, remain 30
    SakuraKallen get 12 damage, lost 3 hp, remain 27
    SakuraKallen get 12 damage, lost 3 hp, remain 24
    SakuraKallen eat onigiri, hp to 49
    SakuraKallen BIG onigiri
    Bronya get 25 element damage, lost 25 hp, remain 30
     - round 5 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 37
    SakuraKallen attack enemy 20
    Bronya get 20 damage, lost 10 hp, remain 20
     - round 6 - 
    Bronya motor bike da!
    SakuraKallen get 23 element damage, lost 23 hp, remain 14
    SakuraKallen BIG onigiri
    Bronya get 25 element damage, lost 25 hp, remain -5
    SakuraKallen win
    ----------
     - round 1 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 88
    SakuraKallen attack enemy 20
    Bronya get 20 damage, lost 10 hp, remain 90
     - round 2 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 76
    SakuraKallen BIG onigiri
    Bronya get 25 element damage, lost 25 hp, remain 65
     - round 3 - 
    Bronya motor bike da!
    SakuraKallen get 61 element damage, lost 61 hp, remain 15
    SakuraKallen attack enemy 20
    Bronya get 20 damage, lost 10 hp, remain 55
     - round 4 - 
    Bronya attack enemy 21
    SakuraKallen get 21 damage, lost 12 hp, remain 3
    Bronya use drill!
    SakuraKallen get 12 damage, lost 3 hp, remain 0
    SakuraKallen get 12 damage, lost 3 hp, remain -3
    SakuraKallen get 12 damage, lost 3 hp, remain -6
    SakuraKallen get 12 damage, lost 3 hp, remain -9
    Bronya win
    ----------
    0.5



```python
winrate(Bronya, Seele, True, 2)
```

     - round 1 - 
    Seele model change!
    Seele attack enemy 18
    Bronya get 18 damage, lost 8 hp, remain 92
    Bronya attack enemy 21
    Seele get 21 damage, lost 0 hp, remain 100
     - round 2 - 
    Seele model change!
    Seele attack enemy 33
    Bronya get 33 damage, lost 23 hp, remain 69
    Bronya attack enemy 21
    Seele get 21 damage, lost 13 hp, remain 88
     - round 3 - 
    Seele model change!
    Seele attack enemy 18
    Bronya get 18 damage, lost 8 hp, remain 61
    Bronya motor bike da!
    Seele get 81 element damage, lost 81 hp, remain 7
     - round 4 - 
    Seele model change!
    Seele attack enemy 33
    Bronya get 33 damage, lost 23 hp, remain 38
    Bronya attack enemy 21
    Seele get 21 damage, lost 13 hp, remain -1
    Bronya win
    ----------
     - round 1 - 
    Seele model change!
    Seele attack enemy 18
    Bronya get 18 damage, lost 8 hp, remain 92
    Bronya attack enemy 21
    Seele get 21 damage, lost 0 hp, remain 100
     - round 2 - 
    Seele model change!
    Seele attack enemy 33
    Bronya get 33 damage, lost 23 hp, remain 69
    Bronya attack enemy 21
    Seele get 21 damage, lost 13 hp, remain 92
     - round 3 - 
    Seele model change!
    Seele attack enemy 18
    Bronya get 18 damage, lost 8 hp, remain 61
    Bronya motor bike da!
    Seele get 65 element damage, lost 65 hp, remain 27
     - round 4 - 
    Seele model change!
    Seele attack enemy 33
    Bronya get 33 damage, lost 23 hp, remain 38
    Bronya attack enemy 21
    Seele get 21 damage, lost 13 hp, remain 22
     - round 5 - 
    Seele model change!
    Seele attack enemy 18
    Bronya get 18 damage, lost 8 hp, remain 30
    Bronya attack enemy 21
    Seele get 21 damage, lost 0 hp, remain 22
     - round 6 - 
    Seele model change!
    Seele attack enemy 33
    Bronya get 33 damage, lost 23 hp, remain 7
    Bronya motor bike da!
    Seele get 29 element damage, lost 29 hp, remain 4
     - round 7 - 
    Seele model change!
    Seele attack enemy 18
    Bronya get 18 damage, lost 8 hp, remain -1
    Seele win
    ----------
    0.5



```python
winrate(Olenyeva, Durandal, True, 2)
```

     - round 1 - 
    Durandal mo yu
    Durandal attack enemy 22
    Olenyeva get 22 damage, lost 12 hp, remain 88
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 92
     - round 2 - 
    Durandal mo yu
    Durandal attack enemy 25
    Olenyeva get 25 damage, lost 15 hp, remain 73
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 84
     - round 3 - 
    Durandal mo yu
    Durandal attack enemy 28
    Olenyeva get 28 damage, lost 18 hp, remain 55
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 76
     - round 4 - 
    Durandal mo yu
    Durandal attack enemy 31
    Olenyeva get 31 damage, lost 21 hp, remain 34
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 68
     - round 5 - 
    Durandal mo yu
    Durandal attack enemy 34
    Olenyeva get 34 damage, lost 24 hp, remain 10
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 60
     - round 6 - 
    Durandal mo yu
    Durandal attack enemy 37
    Olenyeva get 37 damage, lost 27 hp, remain -17
    Olenyeva 96C live water
    Olenyeva become star!
    Durandal get 233 damage, lost 223 hp, remain -163
    Olenyeva win
    ----------
     - round 1 - 
    Durandal mo yu
    Durandal attack enemy 22
    Olenyeva get 22 damage, lost 12 hp, remain 88
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 92
     - round 2 - 
    Durandal mo yu
    Durandal attack enemy 25
    Olenyeva get 25 damage, lost 15 hp, remain 73
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 84
     - round 3 - 
    Durandal mo yu
    Durandal attack enemy 28
    Olenyeva get 28 damage, lost 18 hp, remain 55
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 76
     - round 4 - 
    Durandal mo yu
    Durandal attack enemy 31
    Olenyeva get 31 damage, lost 21 hp, remain 34
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 68
     - round 5 - 
    Durandal mo yu
    Durandal attack enemy 34
    Olenyeva get 34 damage, lost 24 hp, remain 10
    Olenyeva attack enemy 18
    Durandal get 18 damage, lost 8 hp, remain 60
     - round 6 - 
    Durandal mo yu
    Durandal attack enemy 37
    Olenyeva get 37 damage, lost 27 hp, remain -17
    Olenyeva 96C live water
    Olenyeva become star!
    Durandal get 233 damage, lost 223 hp, remain -163
    Olenyeva win
    ----------
    1.0



```python
winrate(Kiana, Durandal, True, 2)
```

     - round 1 - 
    Kiana attack enemy 24
    Durandal get 24 damage, lost 14 hp, remain 86
    Durandal mo yu
    Durandal attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 89
     - round 2 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Durandal get 44 damage, lost 34 hp, remain 52
    Durandal mo yu
    Durandal attack enemy 25
    Kiana get 25 damage, lost 14 hp, remain 75
     - round 3 - 
    Kiana attack enemy 24
    Durandal get 24 damage, lost 14 hp, remain 38
    Durandal mo yu
    Durandal attack enemy 28
    Kiana get 28 damage, lost 17 hp, remain 58
     - round 4 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Durandal get 44 damage, lost 34 hp, remain 4
    Durandal mo yu
    Durandal attack enemy 31
    Kiana get 31 damage, lost 20 hp, remain 38
     - round 5 - 
    Kiana attack enemy 24
    Durandal get 24 damage, lost 14 hp, remain -10
    Kiana win
    ----------
     - round 1 - 
    Kiana attack enemy 24
    Durandal get 24 damage, lost 14 hp, remain 86
    Durandal mo yu
    Durandal attack enemy 22
    Kiana get 22 damage, lost 11 hp, remain 89
     - round 2 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Durandal get 44 damage, lost 34 hp, remain 52
    Durandal mo yu
    Durandal attack enemy 25
    Kiana get 25 damage, lost 14 hp, remain 75
     - round 3 - 
    Kiana attack enemy 24
    Durandal get 24 damage, lost 14 hp, remain 38
    Durandal mo yu
    Durandal attack enemy 28
    Kiana get 28 damage, lost 17 hp, remain 58
     - round 4 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Durandal get 44 damage, lost 34 hp, remain 4
    Durandal mo yu
    Durandal attack enemy 31
    Kiana get 31 damage, lost 20 hp, remain 38
     - round 5 - 
    Kiana attack enemy 24
    Durandal get 24 damage, lost 14 hp, remain -10
    Kiana win
    ----------
    1.0



```python
winrate(Kiana, Olenyeva, True, 2)
```

     - round 1 - 
    Kiana attack enemy 24
    Olenyeva get 24 damage, lost 14 hp, remain 86
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 93
     - round 2 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain 52
    Kiana make herself down
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 86
     - round 3 - 
    Kiana down, do nothing
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 79
     - round 4 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain 18
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 72
     - round 5 - 
    Kiana attack enemy 24
    Olenyeva get 24 damage, lost 14 hp, remain 4
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 65
     - round 6 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain -30
    Olenyeva 96C live water
    Kiana make herself down
    Olenyeva become star!
    Kiana get 233 damage, lost 222 hp, remain -157
    Olenyeva win
    ----------
     - round 1 - 
    Kiana attack enemy 24
    Olenyeva get 24 damage, lost 14 hp, remain 86
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 93
     - round 2 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain 52
    Kiana make herself down
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 86
     - round 3 - 
    Kiana down, do nothing
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 79
     - round 4 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain 18
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 72
     - round 5 - 
    Kiana attack enemy 24
    Olenyeva get 24 damage, lost 14 hp, remain 4
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 65
     - round 6 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain -30
    Olenyeva 96C live water
    Olenyeva become star!
    Kiana get 50 damage, lost 39 hp, remain 26
     - round 7 - 
    Kiana attack enemy 24
    Olenyeva get 24 damage, lost 14 hp, remain 6
    Olenyeva attack enemy 18
    Kiana get 18 damage, lost 7 hp, remain 19
     - round 8 - 
    Kiana attack up by 20
    Kiana attack enemy 44
    Olenyeva get 44 damage, lost 34 hp, remain -28
    Kiana make herself down
    Kiana win
    ----------
    0.5



```python
winrate(Rita, Theresa, True, 2)
```

     - round 1 - 
    Theresa attack enemy 19
    Rita get 19 damage, lost 8 hp, remain 92
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 89
     - round 2 - 
    Theresa attack enemy 15
    Rita get 15 damage, lost 4 hp, remain 88
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 75
     - round 3 - 
    Theresa online kick
    Rita get 16 damage, lost 5 hp, remain 83
    Rita get 16 damage, lost 5 hp, remain 78
    Rita get 16 damage, lost 5 hp, remain 73
    Rita get 16 damage, lost 5 hp, remain 68
    Rita get 16 damage, lost 5 hp, remain 63
    Theresa blood judas is the most cute
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 64
     - round 4 - 
    Theresa attack enemy 11
    Rita get 11 damage, lost 5 hp, remain 58
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 54
     - round 5 - 
    Theresa attack enemy 11
    Rita get 11 damage, lost 3 hp, remain 55
    Theresa blood judas is the most cute
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 43
     - round 6 - 
    Theresa attack enemy 7
    Rita get 7 damage, lost 3 hp, remain 51
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 32
     - round 7 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 50
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 18
     - round 8 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 48
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 8
     - round 9 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 47
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain -3
    Rita win
    ----------
     - round 1 - 
    Theresa attack enemy 19
    Rita get 19 damage, lost 8 hp, remain 92
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 89
     - round 2 - 
    Theresa attack enemy 15
    Rita get 15 damage, lost 4 hp, remain 88
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 78
     - round 3 - 
    Theresa online kick
    Rita get 16 damage, lost 5 hp, remain 83
    Rita get 16 damage, lost 5 hp, remain 78
    Rita get 16 damage, lost 5 hp, remain 73
    Rita get 16 damage, lost 5 hp, remain 68
    Rita get 16 damage, lost 5 hp, remain 63
    Theresa blood judas is the most cute
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 67
     - round 4 - 
    Theresa attack enemy 7
    Rita get 7 damage, lost 1 hp, remain 62
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 57
     - round 5 - 
    Theresa attack enemy 7
    Rita get 7 damage, lost 0 hp, remain 61
    Theresa blood judas is the most cute
    Rita maid care
    Rita attack enemy 23
    Theresa get 23 damage, lost 11 hp, remain 46
     - round 6 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 60
    Theresa blood judas is the most cute
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 32
     - round 7 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 58
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 18
     - round 8 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 56
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain 8
     - round 9 - 
    Theresa attack enemy 3
    Rita get 3 damage, lost 1 hp, remain 54
    Rita attack enemy 26
    Theresa get 26 damage, lost 14 hp, remain -6
    Rita win
    ----------
    1.0



```python
winrate(Rita, Fuhua, True, 2)
```

     - round 1 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 89
    Fuhua attack enemy 17
    Rita get 17 element damage, lost 17 hp, remain 83
     - round 2 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 78
    Fuhua attack enemy 17
    Rita get 17 element damage, lost 17 hp, remain 66
     - round 3 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 67
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain 48
     - round 4 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 60
    Fuhua attack enemy 17
    Rita get 17 damage, lost 3 hp, remain 44
     - round 5 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 49
    Fuhua attack enemy 17
    Rita get 17 damage, lost 3 hp, remain 40
     - round 6 - 
    Rita maid care
    Rita attack enemy 23
    Fuhua get 23 damage, lost 8 hp, remain 41
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain 22
     - round 7 - 
    Rita maid care
    Rita missed!
    Fuhua attack enemy 9
    Rita get 5 element damage, lost 5 hp, remain 17
     - round 8 - 
    Rita maid care
    Rita missed!
    Fuhua attack enemy 5
    Rita get 5 damage, lost 0 hp, remain 17
     - round 9 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 34
    Fuhua attack enemy 5
    Rita get 5 damage, lost 0 hp, remain 17
     - round 10 - 
    Rita missed!
    Fuhua attack enemy 5
    Rita get 3 element damage, lost 3 hp, remain 14
     - round 11 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 23
    Fuhua attack enemy 5
    Rita get 3 element damage, lost 3 hp, remain 11
     - round 12 - 
    Rita missed!
    Fuhua attack enemy 5
    Rita get 5 damage, lost 0 hp, remain 11
     - round 13 - 
    Rita missed!
    Fuhua attack enemy 5
    Rita get 5 damage, lost 0 hp, remain 11
     - round 14 - 
    Rita maid care
    Rita missed!
    Fuhua attack enemy 1
    Rita get 0 element damage, lost 0 hp, remain 10
     - round 15 - 
    Rita maid care
    Rita missed!
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain -7
    Fuhua win
    ----------
     - round 1 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 89
    Fuhua attack enemy 17
    Rita get 17 element damage, lost 17 hp, remain 83
     - round 2 - 
    Rita maid care
    Rita attack enemy 23
    Fuhua get 23 damage, lost 8 hp, remain 81
    Fuhua attack enemy 13
    Rita get 13 element damage, lost 13 hp, remain 70
     - round 3 - 
    Rita maid care
    Rita attack enemy 23
    Fuhua get 23 damage, lost 8 hp, remain 73
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain 52
     - round 4 - 
    Rita maid care
    Rita attack enemy 23
    Fuhua get 23 damage, lost 8 hp, remain 69
    Fuhua attack enemy 5
    Rita get 5 damage, lost 0 hp, remain 52
     - round 5 - 
    Rita maid care
    Rita missed!
    Fuhua attack enemy 1
    Rita get 1 damage, lost 0 hp, remain 52
     - round 6 - 
    Rita missed!
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain 34
     - round 7 - 
    Rita missed!
    Fuhua attack enemy 1
    Rita get 0 element damage, lost 0 hp, remain 33
     - round 8 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 62
    Fuhua attack enemy 1
    Rita get 1 damage, lost 0 hp, remain 33
     - round 9 - 
    Rita maid care
    Rita attack enemy 23
    Fuhua get 23 damage, lost 8 hp, remain 54
    Fuhua attack enemy 0
    Rita get 0 damage, lost 0 hp, remain 33
     - round 10 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 43
    Fuhua attack enemy 0
    Rita get 0 element damage, lost 0 hp, remain 33
     - round 11 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 32
    Fuhua attack enemy 0
    Rita get 0 element damage, lost 0 hp, remain 33
     - round 12 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 25
    Fuhua attack enemy 0
    Rita get 0 damage, lost 0 hp, remain 33
     - round 13 - 
    Rita attack enemy 26
    Fuhua get 26 damage, lost 11 hp, remain 14
    Fuhua attack enemy 0
    Rita get 0 damage, lost 0 hp, remain 33
     - round 14 - 
    Rita missed!
    Fuhua attack enemy 0
    Rita get 0 element damage, lost 0 hp, remain 33
     - round 15 - 
    Rita maid care
    Rita attack enemy 23
    Fuhua get 23 damage, lost 8 hp, remain 6
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain 15
     - round 16 - 
    Rita missed!
    Fuhua attack enemy 0
    Rita get 0 damage, lost 0 hp, remain 15
     - round 17 - 
    Rita maid care
    Rita missed!
    Fuhua attack enemy 0
    Rita get 0 damage, lost 0 hp, remain 15
     - round 18 - 
    Rita missed!
    Fuhua ink
    Rita get 18 element damage, lost 18 hp, remain -2
    Fuhua win
    ----------
    0.0



```python
winrate(Theresa, Fuhua, True, 2)
```

     - round 1 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 4 hp, remain 96
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 83
     - round 2 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 4 hp, remain 92
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 66
     - round 3 - 
    Theresa online kick
    Fuhua get 16 damage, lost 1 hp, remain 91
    Fuhua get 16 damage, lost 1 hp, remain 90
    Fuhua get 16 damage, lost 1 hp, remain 89
    Fuhua get 16 damage, lost 1 hp, remain 88
    Fuhua get 16 damage, lost 1 hp, remain 87
    Fuhua ink
    Theresa get 18 element damage, lost 18 hp, remain 48
     - round 4 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 4 hp, remain 83
    Theresa blood judas is the most cute
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 31
     - round 5 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 9 hp, remain 74
    Theresa blood judas is the most cute
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 14
     - round 6 - 
    Theresa online kick
    Fuhua get 16 damage, lost 11 hp, remain 63
    Fuhua get 16 damage, lost 11 hp, remain 52
    Fuhua get 16 damage, lost 11 hp, remain 41
    Fuhua get 16 damage, lost 11 hp, remain 30
    Fuhua get 16 damage, lost 11 hp, remain 19
    Fuhua ink
    Theresa get 18 element damage, lost 18 hp, remain -4
    Fuhua win
    ----------
     - round 1 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 4 hp, remain 96
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 83
     - round 2 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 4 hp, remain 92
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 66
     - round 3 - 
    Theresa online kick
    Fuhua get 16 damage, lost 1 hp, remain 91
    Fuhua get 16 damage, lost 1 hp, remain 90
    Fuhua get 16 damage, lost 1 hp, remain 89
    Fuhua get 16 damage, lost 1 hp, remain 88
    Fuhua get 16 damage, lost 1 hp, remain 87
    Fuhua ink
    Theresa get 18 element damage, lost 18 hp, remain 48
     - round 4 - 
    Theresa missed!
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 31
     - round 5 - 
    Theresa attack enemy 19
    Fuhua get 19 damage, lost 4 hp, remain 83
    Theresa blood judas is the most cute
    Fuhua attack enemy 17
    Theresa get 17 element damage, lost 17 hp, remain 14
     - round 6 - 
    Theresa online kick
    Fuhua get 16 damage, lost 6 hp, remain 77
    Fuhua get 16 damage, lost 6 hp, remain 71
    Fuhua get 16 damage, lost 6 hp, remain 65
    Fuhua get 16 damage, lost 6 hp, remain 59
    Fuhua get 16 damage, lost 6 hp, remain 53
    Fuhua ink
    Theresa get 18 element damage, lost 18 hp, remain -4
    Fuhua win
    ----------
    0.0



```python
winrate(Mei, Himeko, True, 2)
```

     - round 1 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 87
    Himeko attack enemy 23
    Mei get 23 damage, lost 11 hp, remain 89
     - round 2 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain 72
    Mei make enemy down
    Himeko down, do nothing
     - round 3 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 59
    Himeko attack enemy 46
    Mei get 46 damage, lost 34 hp, remain 55
     - round 4 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain 44
    Himeko missed!
     - round 5 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 31
    Himeko missed!
     - round 6 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain 16
    Himeko missed!
     - round 7 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 3
    Himeko missed!
     - round 8 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain -12
    Mei make enemy down
    Mei win
    ----------
     - round 1 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 87
    Himeko attack enemy 23
    Mei get 23 damage, lost 11 hp, remain 89
     - round 2 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain 72
    Mei make enemy down
    Himeko down, do nothing
     - round 3 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 59
    Mei make enemy down
    Himeko down, do nothing
     - round 4 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain 44
    Himeko missed!
     - round 5 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 31
    Himeko missed!
     - round 6 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain 16
    Mei make enemy down
    Himeko down, do nothing
     - round 7 - 
    Mei attack enemy 22
    Himeko get 22 damage, lost 13 hp, remain 3
    Himeko missed!
     - round 8 - 
    Mei use dragon blaze!
    Himeko get 15 element damage, lost 15 hp, remain -12
    Mei win
    ----------
    1.0



```python
winrate(Mei, Raven, True, 2)
```

     - round 1 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 92
    Raven not only hurts you
    Raven attack enemy 23
    Mei get 23 damage, lost 13 hp, remain 86
     - round 2 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 77
    Raven attack enemy 23
    Mei get 23 damage, lost 13 hp, remain 72
     - round 3 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 69
    Raven MY ISLAND
    Mei get 16 damage, lost 5 hp, remain 67
    Mei get 16 damage, lost 5 hp, remain 62
    Mei get 16 damage, lost 5 hp, remain 57
    Mei get 16 damage, lost 5 hp, remain 52
    Mei get 16 damage, lost 5 hp, remain 47
    Mei get 16 damage, lost 5 hp, remain 42
    Mei get 16 damage, lost 5 hp, remain 37
     - round 4 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 54
    Mei make enemy down
    Raven down, do nothing
     - round 5 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 46
    Raven attack enemy 23
    Mei get 23 damage, lost 13 hp, remain 23
     - round 6 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 31
    Raven MY ISLAND
    Mei get 16 damage, lost 5 hp, remain 18
    Mei get 16 damage, lost 5 hp, remain 13
    Mei get 16 damage, lost 5 hp, remain 8
    Mei get 16 damage, lost 5 hp, remain 3
    Mei get 16 damage, lost 5 hp, remain -1
    Mei get 16 damage, lost 5 hp, remain -6
    Mei get 16 damage, lost 5 hp, remain -11
    Raven win
    ----------
     - round 1 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 92
    Raven attack enemy 23
    Mei get 23 damage, lost 11 hp, remain 89
     - round 2 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 77
    Raven attack enemy 23
    Mei get 23 damage, lost 11 hp, remain 78
     - round 3 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 69
    Raven MY ISLAND
    Mei get 16 damage, lost 4 hp, remain 74
    Mei get 16 damage, lost 4 hp, remain 70
    Mei get 16 damage, lost 4 hp, remain 66
    Mei get 16 damage, lost 4 hp, remain 62
    Mei get 16 damage, lost 4 hp, remain 58
    Mei get 16 damage, lost 4 hp, remain 54
    Mei get 16 damage, lost 4 hp, remain 50
     - round 4 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 54
    Raven not only hurts you
    Raven attack enemy 23
    Mei get 23 damage, lost 13 hp, remain 36
     - round 5 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 46
    Mei make enemy down
    Raven down, do nothing
     - round 6 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 31
    Mei make enemy down
    Raven MY ISLAND
    Mei get 16 damage, lost 5 hp, remain 31
    Mei get 16 damage, lost 5 hp, remain 26
    Mei get 16 damage, lost 5 hp, remain 21
    Mei get 16 damage, lost 5 hp, remain 16
    Mei get 16 damage, lost 5 hp, remain 11
    Mei get 16 damage, lost 5 hp, remain 6
    Mei get 16 damage, lost 5 hp, remain 1
     - round 7 - 
    Mei attack enemy 22
    Raven get 22 damage, lost 8 hp, remain 23
    Mei make enemy down
    Raven down, do nothing
     - round 8 - 
    Mei use dragon blaze!
    Raven get 15 element damage, lost 15 hp, remain 8
    Raven attack enemy 23
    Mei get 23 damage, lost 13 hp, remain -12
    Raven win
    ----------
    0.0



```python
winrate(Himeko, Raven, True, 2)
```

     - round 1 - 
    Raven not only hurts you
    Raven attack enemy 23
    Himeko get 23 damage, lost 17 hp, remain 82
    Himeko attack enemy 23
    Raven get 23 damage, lost 9 hp, remain 91
     - round 2 - 
    Raven not only hurts you
    Raven attack enemy 23
    Himeko get 23 damage, lost 17 hp, remain 65
    Himeko attack enemy 46
    Raven get 46 damage, lost 32 hp, remain 59
     - round 3 - 
    Raven MY ISLAND
    Himeko get 16 damage, lost 8 hp, remain 56
    Himeko get 16 damage, lost 8 hp, remain 47
    Himeko get 16 damage, lost 8 hp, remain 38
    Himeko get 16 damage, lost 8 hp, remain 30
    Himeko get 16 damage, lost 8 hp, remain 21
    Himeko get 16 damage, lost 8 hp, remain 12
    Himeko get 16 damage, lost 8 hp, remain 3
    Himeko attack enemy 46
    Raven get 46 damage, lost 32 hp, remain 27
     - round 4 - 
    Raven attack enemy 23
    Himeko get 23 damage, lost 17 hp, remain -13
    Raven win
    ----------
     - round 1 - 
    Raven attack enemy 23
    Himeko get 23 damage, lost 14 hp, remain 86
    Himeko attack enemy 23
    Raven get 23 damage, lost 9 hp, remain 91
     - round 2 - 
    Raven attack enemy 23
    Himeko get 23 damage, lost 14 hp, remain 72
    Himeko attack enemy 46
    Raven get 46 damage, lost 32 hp, remain 59
     - round 3 - 
    Raven MY ISLAND
    Himeko get 16 damage, lost 7 hp, remain 65
    Himeko get 16 damage, lost 7 hp, remain 58
    Himeko get 16 damage, lost 7 hp, remain 51
    Himeko get 16 damage, lost 7 hp, remain 44
    Himeko get 16 damage, lost 7 hp, remain 37
    Himeko get 16 damage, lost 7 hp, remain 30
    Himeko get 16 damage, lost 7 hp, remain 23
    Himeko missed!
     - round 4 - 
    Raven attack enemy 23
    Himeko get 23 damage, lost 14 hp, remain 9
    Himeko missed!
     - round 5 - 
    Raven not only hurts you
    Raven attack enemy 23
    Himeko get 23 damage, lost 17 hp, remain -8
    Raven win
    ----------
    0.0


# 7.30


```python
winrate(Kiana, Mei)
```

    0.48637



```python
winrate(SakuraKallen, Seele)
```

    0.2299



```python
winrate(Bronya, SakuraKallen)
```

    0.76701



```python
winrate(Bronya, Seele)
```

    0.63717


# 7.31


```python
winrate(Olenyeva, Durandal)
```

    0.42009



```python
winrate(Kiana, Durandal)
```

    1.0



```python
winrate(Kiana, Olenyeva)
```

    0.50337


# 8.1


```python
winrate(Rita, Theresa)
```

    0.667



```python
winrate(Rita, Fuhua)
```

    0.25394



```python
winrate(Theresa, Fuhua)
```

    0.36601


# 8.2


```python
winrate(Mei, Himeko)
```

    0.72326



```python
winrate(Mei, Raven)
```

    0.12596



```python
winrate(Himeko, Raven)
```

    0.11105

