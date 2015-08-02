

# Once Upon a Time in America #

Когда-то, году в 2002-м, на мой компьютер попала интересная игрушка под названием Amazing Blocks. Игра, так сказать, класса тетриса (подробное описание геймплея приведено ниже); она очень полюбилась моей маме, которая играла в эту игру часами. Однако был досадный недостаток: после, кажется, 10 запусков игра начинала требовать регистрацию, которая, что удивительно, была бесплатной, но через интернет, что, само собой, являлось непреодолимым препятствием, поскольку интернета-то никакого тогда в наших краях в глаза не видывали, хоть и слыхали, что есть такая штука. Приходилось постоянно переустанавливать.

Через года три, когда интернет уже провели, а игра успела стать shareware и начать просить за регистрацию сколько-то денег, я попробовал её зарегистрировать, однако сайт производителя был к тому времени скорее мёртв, чем жив, и, по-видимому, остаётся таким и по сей день. В интернете легко находится shareware-версия игры, множество, не побоюсь этого слова, кейгенов, являющихся на самом деле троянами, и ни одной возможности зарегистрировать игру, чтобы мама могла в неё играть уже совсем на другом компьютере. В какой-то момент я подумал: а почему бы просто самому не сделать аналогичную игру и решить тем самым проблему на корню? Заодно из этого может получится какой-никакой hello-world по разработке простой игры для ПК в современных условиях — который я и предлагаю вниманию читателей.

![http://besisland.name/img/blog/impressive-solids-habr-1.png](http://besisland.name/img/blog/impressive-solids-habr-1.png)

Итак, что же за игру мы будем делать? Суть такова. В прямоугольном стакане 7×13 падает горизонтальная палка, состоящая из 3 цветных блоков (всего есть 5 цветов). Во время движения её можно двигать вправо-влево, а также менять блоки местами в порядке ротации справа налево (красный, зелёный, синий → зелёный, синий, красный). Как только палка коснётся пола стакана или же какого-нибудь из находящихся в стакане неподвижных блоков, ею больше управлять нельзя. Блоки, составляющие палку, продолжают падение отдельно до тех пор, пока не станут на неподвижный блок или пол стакана. После этого проверяется, не получилась ли в стакане горизонтальная, вертикальная или диагональная линия из трёх или более блоков одного цвета; такие линии уничтожаются. Если сверху уничтоженной линии были блоки, они сползают вниз на образовавшееся пустое место, после этого снова происходит уничтожение образовавшихся линий. Когда всё устаканилось, сверху начинает падать новая палка. За выстраивание уничтожаемых линий игрок получает очки. Игра заканчивается, когда стакан заполнен доверху.

Технологии. Игру будем делать на C# (давно хотел посмотреть, что же это такое), OpenGL (DirectX работает только под Windows, а я больше люблю Linux), Mercurial для контроля версий (писать код без VCS — неуважение к себе).

Игра будет называться Impressive Solids.

# Inception #

Разработку под Windows будем вести в [Microsoft Visual C# 2010 Express](http://www.microsoft.com/visualstudio/en-us/products/2010-editions/visual-csharp-express) (распространяется бесплатно). Также нам понадобится [TortoiseHg](http://tortoisehg.bitbucket.org/) — Windows-клиент системы контроля версий Mercurial. Под системы на базе Linux будем использовать MonoDevelop и консольный hg.

Для подключения OpenGL задействуем binding [OpenTK](http://www.opentk.com/). Нужно скачать свежий [nightly build](http://sourceforge.net/projects/opentk/files/opentk/nightly/) (на момент написания статьи: 2011-12-03).

Создаём в Visual C# Express новый empty project под названием ImpressiveSolids. Сохраняем. Затем открываем директорию с проектом, вызываем для неё контекстное меню и выбираем TortoiseHg → Create Repository Here. Отмечаем пункты создания файла .hgignore и открытия workbench после инициализации.

Открываем в Visual C# Express файл .hgignore и записываем в него следующие строки. Это нужно для того, чтобы система контроля версий не учитывала ненужные бинарные файлы.

```
syntax: glob
*.suo
*.pidb
ImpressiveSolids/bin/*
ImpressiveSolids/obj/*
```

Внутри директории solution (не проекта; там, где лежит .hgignore) создаём поддиректорию OpenTK и копируем в неё файлы `OpenTK*.dll` и `OpenTK*.dll.config` из директории opentk\Binaries\OpenTK\Release\ в архиве OpenTK.

В Visual C# Express контекстное меню References → Add Reference → Browse. Выбираем ../OpenTK/OpenTK.dll. Кроме того, нужно добавить reference на System.Drawing с вкладки .NET.

Создаём новый класс `Game`. Это главный класс программы, в нём находится точка входа, а сам он — наследник `OpenTK.GameWindow` и отвечает за обновление состояния игры (`OnUpdateFrame`) и перерисовку (`OnRenderFrame`). Сейчас это будет просто чёрное окно.

```
using System;
using OpenTK;
using OpenTK.Graphics;
using OpenTK.Graphics.OpenGL;

namespace ImpressiveSolids {
    class Game : GameWindow {
        [STAThread]
        static void Main() {
            using (var Game = new Game()) {
                Game.Run(30);
            }
        }

        public Game()
            : base(700, 500, GraphicsMode.Default, "Impressive Solids") {
            VSync = VSyncMode.On;
        }

        protected override void OnLoad(EventArgs E) {
            base.OnLoad(E);
        }

        protected override void OnResize(EventArgs E) {
            base.OnResize(E);
            GL.Viewport(ClientRectangle.X, ClientRectangle.Y, ClientRectangle.Width, ClientRectangle.Height);
        }

        protected override void OnUpdateFrame(FrameEventArgs E) {
            base.OnUpdateFrame(E);
        }

        protected override void OnRenderFrame(FrameEventArgs E) {
            base.OnRenderFrame(E);

            GL.ClearColor(Color4.Black);
            GL.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

            SwapBuffers();
        }
    }
}
```

Заходим в свойства проекта (Project → ImpressiveSolids Properties) и указываем Target framework: .NET Framework 2.0; Output type: Windows Application; Startup object: ImpressiveSolids.Game.

Можно сохранять и запускать, должно появиться чёрное окно размером 700×500 с заголовком «Impressive Solids».

Если всё прошло гладко, идём в TortoiseHg Workbench и коммитим всё с пометкой «Initial game window».

# The Fall #

Реализуем управляемое падение палки. Для этого прежде всего нужно задать модель текущего состояния палки. Во-первых, позиция. По умолчанию — сверху по центру стакана. Будем считать, что (0; 0) соответствует верхнему левому углу стакана. Нужно, кстати, задать его размеры `MapWidth`, `MapHeight`. Цвета блоков, составляющих палку, будем хранить как массив целых чисел; зададим количество возможных цветов `ColorsCount` и договоримся, что цвет обозначается целочисленным значением от `0` до `ColorsCount − 1`.

Добавим в класс `Game` метод `New` и вызовем его из `OnLoad`. В этом методе реализуем построение палки из блоков случайных цветов.

```
private Random Rand;

private const int MapWidth = 7;
private const int MapHeight = 13;

private const int StickLength = 3;
private int[] StickColors;
private Vector2 StickPosition;

private const int ColorsCount = 5;

protected override void OnLoad(EventArgs E) {
    base.OnLoad(E);
    New();
}

private void New() {
    Rand = new Random();
    StickColors = new int[StickLength];
    for (var i = 0; i < StickLength; i++) {
        StickColors[i] = Rand.Next(ColorsCount);
    }
    StickPosition.X = (float)Math.Floor((MapWidth - StickLength) / 2d);
    StickPosition.Y = 0;
}
```

Попробуем отобразить нашу палку на экране, пока в самом примитивном варианте (блок изображаем цветным прямоугольником, стакан начинаем прямо в верхнем левом углу окна). Внесём в код некоторые изменения.

```
private const int NominalWidth = 700;
private const int NominalHeight = 500;

private float ProjectionWidth;
private float ProjectionHeight;

private const int SolidSize = 35;

private Color4[] Colors = {Color4.PaleVioletRed, Color4.LightSeaGreen, Color4.CornflowerBlue, Color4.RosyBrown, Color4.LightGoldenrodYellow};

public Game()
    : base(NominalWidth, NominalHeight, GraphicsMode.Default, "Impressive Solids") {
    VSync = VSyncMode.On;
}

protected override void OnResize(EventArgs E) {
    base.OnResize(E);
    GL.Viewport(ClientRectangle.X, ClientRectangle.Y, ClientRectangle.Width, ClientRectangle.Height);

    ProjectionWidth = NominalWidth;
    ProjectionHeight = (float)ClientRectangle.Height / (float)ClientRectangle.Width * ProjectionWidth;
    if (ProjectionHeight < NominalHeight) {
        ProjectionHeight = NominalHeight;
        ProjectionWidth = (float)ClientRectangle.Width / (float)ClientRectangle.Height * ProjectionHeight;
    }
}

protected override void OnRenderFrame(FrameEventArgs E) {
    base.OnRenderFrame(E);

    GL.ClearColor(Color4.Black);
    GL.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

    var Projection = Matrix4.CreateOrthographic(-ProjectionWidth, -ProjectionHeight, -1, 1);
    GL.MatrixMode(MatrixMode.Projection);
    GL.LoadMatrix(ref Projection);
    GL.Translate(ProjectionWidth / 2, -ProjectionHeight / 2, 0);

    var Modelview = Matrix4.LookAt(Vector3.Zero, Vector3.UnitZ, Vector3.UnitY);
    GL.MatrixMode(MatrixMode.Modelview);
    GL.LoadMatrix(ref Modelview);

    GL.Begin(BeginMode.Quads);

    for (var i = 0; i < StickLength; i++) {
        RenderSolid(StickPosition.X + i, StickPosition.Y, StickColors[i]);
    }

    GL.End();

    SwapBuffers();
}

private void RenderSolid(float X, float Y, int Color) {
    GL.Color4(Colors[Color]);
    GL.Vertex2(X * SolidSize, Y * SolidSize);
    GL.Vertex2((X + 1) * SolidSize, Y * SolidSize);
    GL.Vertex2((X + 1) * SolidSize, (Y + 1) * SolidSize);
    GL.Vertex2(X * SolidSize, (Y + 1) * SolidSize);
}
```

Ухищрения с `Nominal/Projection Width/Height` понадобились для того, чтобы изображение масштабировалось при изменении размеров окна, но в то же время пропорции не искажались.

Теперь сделаем наконец, чтобы палка падала и чтобы работали клавиши ←, →, ↑ (ротация цветов).

```
public Game()
    : base(NominalWidth, NominalHeight, GraphicsMode.Default, "Impressive Solids") {
    VSync = VSyncMode.On;
    Keyboard.KeyDown += new EventHandler<KeyboardKeyEventArgs>(OnKeyDown);
}

protected override void OnUpdateFrame(FrameEventArgs E) {
    base.OnUpdateFrame(E);
    StickPosition.Y += 0.02f;
}

protected void OnKeyDown(object Sender, KeyboardKeyEventArgs E) {
    if (Key.Left == E.Key) {
        --StickPosition.X;
    } else if (Key.Right == E.Key) {
        ++StickPosition.X;
    } else if (Key.Up == E.Key) {
        var T = StickColors[0];
        for (var i = 0; i < StickLength - 1; i++) {
            StickColors[i] = StickColors[i + 1];
        }
        StickColors[StickLength - 1] = T;
    }
}
```

Коммитим все изменения: «The stick, falling and controllable».

Как видим, пока нет проверки на выход за границы стакана. Исправим это упущение в дальнейшем.

# A Map of the World #

Займёмся ситуацией, когда палка опустилась на пол или на уже присутствующие в стакане блоки. Пока не будем разбираться с последующим неконтролируемым падением блоков и уничтожением линий, а сделаем просто, чтобы составляющие палку блоки застывали на месте (даже повисая в воздухе) и начинала падать следующая палка.

Смоделируем состояние стакана в виде двумерного массива целых чисел. Координаты будут соответствовать клетчатой сетке стакана, значениями будет цвет блока в данной клетке — или отрицательное число, если в клетке пусто.

Здесь уже будет необходимо ввести проверку на выход палки за границы стакана, иначе возникнут обращения к массиву по несуществующим индексам.

```
private int[,] Map;

private void New() {
    Rand = new Random();

    Map = new int[MapWidth, MapHeight];
    for (var X = 0; X < MapWidth; X++) {
        for (var Y = 0; Y < MapHeight; Y++) {
            Map[X, Y] = -1;
        }
    }

    StickColors = new int[StickLength];
    GenerateNextStick();
}

private void GenerateNextStick() {
    for (var i = 0; i < StickLength; i++) {
        StickColors[i] = Rand.Next(ColorsCount);
    }
    StickPosition.X = (float)Math.Floor((MapWidth - StickLength) / 2d);
    StickPosition.Y = 0;
}

protected override void OnUpdateFrame(FrameEventArgs E) {
    base.OnUpdateFrame(E);

    StickPosition.Y += 0.02f;

    var FellOnFloor = (StickPosition.Y >= MapHeight - 1);

    var FellOnBlock = false;
    if (!FellOnFloor) {
        var Y = (int)Math.Floor(StickPosition.Y + 1);
        for (var i = 0; i < StickLength; i++) {
            var X = (int)StickPosition.X + i;
            if (Map[X, Y] >= 0) {
                FellOnBlock = true;
                break;
            }
        }
    }

    if (FellOnFloor || FellOnBlock) {
        var Y = (int)Math.Floor(StickPosition.Y);
        for (var i = 0; i < StickLength; i++) {
            var X = (int)StickPosition.X + i;
            Map[X, Y] = StickColors[i];
        }
        GenerateNextStick();
    }
}

protected void OnKeyDown(object Sender, KeyboardKeyEventArgs E) {
    if ((Key.Left == E.Key) && (StickPosition.X > 0)) {
        --StickPosition.X;
    } else if ((Key.Right == E.Key) && (StickPosition.X + StickLength < MapWidth)) {
        ++StickPosition.X;
    } else if (Key.Up == E.Key) {
        // . . .
    }
}

protected override void OnRenderFrame(FrameEventArgs E) {
    // . . .

    GL.Begin(BeginMode.Quads);

    for (var X = 0; X < MapWidth; X++) {
        for (var Y = 0; Y < MapHeight; Y++) {
            if (Map[X, Y] >= 0) {
                RenderSolid(X, Y, Map[X, Y]);
            }
        }
    }

    for (var i = 0; i < StickLength; i++) {
        RenderSolid(StickPosition.X + i, StickPosition.Y, StickColors[i]);
    }

    GL.End();

    SwapBuffers();
}
```

Теперь можно быстро набросать блоков до самого верха, как в старом добром «Тетрисе». Для тестирования можно увеличить скорость падения, заменив `0.02f` на `0.2f`, а вообще надо будет сделать возможность ускорения по нажатию клавиши ↓.

Не забываем коммитить изменения в репозиторий Mercurial: «Fixing blocks after the stick fell».

# Double Impact #

Следующее, что нам нужно сделать — это чтобы блоки не зависали в воздухе, а продолжали падать вниз, пока не упрутся. В этот период времени палки на экране нет, управлять ничем нельзя. В связи с этим введём в игру понятие состояния.

Игра в каждый момент времени находится в одном из следующих состояний:

  1. Падает очередная палка, ею можно управлять. Когда начинается новая игра, включается это состояние.
  1. Неуправляемое падение блоков, уничтожение выстроившихся линий. Это состояние включается после того, как палка коснулась какого-нибудь блока. Заканчивается тогда, когда все блоки стоят неподвижно и уничтожимых линий нет. Если весь верхний ряд стакана свободен, то игра продолжается в состоянии № 1; иначе игра завершается (состояние № 3).
  1. Игра окончена, ничего не происходит. Игрок может начать новую игру (скажем, нажав некую кнопку).

Сделаем соответствующие объявления в коде.

```
private enum GameStateEnum {
    Fall,
    Impact,
    GameOver
}
private GameStateEnum GameState;

private void New() {
    // . . .
    GenerateNextStick();
    GameState = GameStateEnum.Fall;
}

protected override void OnUpdateFrame(FrameEventArgs E) {
    base.OnUpdateFrame(E);

    if (GameStateEnum.Fall == GameState) {
        StickPosition.Y += 0.2f;

        // . . .

        if (FellOnFloor || FellOnBlock) {
            var Y = (int)Math.Floor(StickPosition.Y);
            for (var i = 0; i < StickLength; i++) {
                var X = (int)StickPosition.X + i;
                Map[X, Y] = StickColors[i];
            }
            GameState = GameStateEnum.Impact;
        }
    } else if (GameStateEnum.Impact == GameState) {
        var Stabilized = true;
        // TODO падение блоков

        if (Stabilized) {
            GenerateNextStick();
            GameState = GameStateEnum.Fall;
        }
    }
}
```

Чтобы изобразить плавное падение блоков и не усложнять при этом модель стакана (`Map`), прибегнем к хитрости. Пусть блок, который падает из клетки (X; Y) в клетку (X; Y + 1) — а куда ещё ему падать? — числится в клетке (X; Y) вплоть до момента окончательного попадания в нижнюю клетку; а дополнительно будем хранить дробное смещение блока по вертикали, которое будет постепенно увеличиваться, пока не превысит единицу. Т. е. реальные координаты блока будут не (X; Y), а (X; Y + Δ), это надо будет учесть в `OnRenderFrame`.

```
private const float FallSpeed = 0.2f;

private float[,] ImpactFallOffset;

private void New() {
    // . . .
    ImpactFallOffset = new float[MapWidth, MapHeight];
}

protected override void OnUpdateFrame(FrameEventArgs E) {
    base.OnUpdateFrame(E);

    if (GameStateEnum.Fall == GameState) {
        StickPosition.Y += FallSpeed;

        // . . .
    } else if (GameStateEnum.Impact == GameState) {
        var Stabilized = true;
        for (var X = 0; X < MapWidth; X++) {
            for (var Y = MapHeight - 2; Y >= 0; Y--) {
                if ((Map[X, Y] >= 0) && ((Map[X, Y + 1] < 0) || (ImpactFallOffset[X, Y + 1] > 0))) {
                    Stabilized = false;
                    ImpactFallOffset[X, Y] += FallSpeed;
                    if (ImpactFallOffset[X, Y] >= 1) {
                        Map[X, Y + 1] = Map[X, Y];
                        Map[X, Y] = -1;
                        ImpactFallOffset[X, Y] = 0;
                    }
                }
            }
        }

        if (Stabilized) {
            GenerateNextStick();
            GameState = GameStateEnum.Fall;
        }
    }
}

protected override void OnRenderFrame(FrameEventArgs E) {
    // . . .

    GL.Begin(BeginMode.Quads);

    for (var X = 0; X < MapWidth; X++) {
        for (var Y = 0; Y < MapHeight; Y++) {
            if (Map[X, Y] >= 0) {
                RenderSolid(X, Y + ImpactFallOffset[X, Y], Map[X, Y]);
            }
        }
    }

    if (GameStateEnum.Fall == GameState) {
        for (var i = 0; i < StickLength; i++) {
            RenderSolid(StickPosition.X + i, StickPosition.Y, StickColors[i]);
        }
    }

    GL.End();

    SwapBuffers();
}
```

Чтобы падать могла сразу целая колонна из блоков, под которыми образовалась дырка, клетки карты перебираем снизу вверх (нижний блок увлекает за собой верхний) и учитываем не только пустоту клетки, но и статус `ImpactFallOffset` (потому что положительное число означает, что этот блок падает вниз).

Коммитим с пометкой: «Blocks fall on impact until stabilized».

# Weapon of Mass Destruction #

Пришла пора наконец заняться основным элементом геймплея: уничтожением выстроенных линий одного цвета. Зададим минимальную длину линии. Достаточно пройти по всем клеткам, в которых есть блоки, и от каждой попытаться построить линию в одном из четырёх возможных направлений (горизонталь, вертикаль и две диагонали). Если обнаружена линия одного цвета достаточной длины, каждый составляющий её блок заносим в стек. После того, как проверена вся карта, уничтожаем все блоки, занесённые в стек, и помечаем положение как нестабильное.

```
private const int DestroyableLength = 3;
private Stack<Vector2> Destroyables = new Stack<Vector2>();

protected override void OnUpdateFrame(FrameEventArgs E) {
    // . . .
    } else if (GameStateEnum.Impact == GameState) {
        // . . .

        if (Stabilized) {
            Destroyables.Clear();

            for (var X = 0; X < MapWidth; X++) {
                for (var Y = 0; Y < MapHeight; Y++) {
                    CheckDestroyableLine(X, Y, 1, 0);
                    CheckDestroyableLine(X, Y, 0, 1);
                    CheckDestroyableLine(X, Y, 1, 1);
                    CheckDestroyableLine(X, Y, 1, -1);
                }
            }

            if (Destroyables.Count > 0) {
                foreach (var Coords in Destroyables) {
                    Map[(int)Coords.X, (int)Coords.Y] = -1;
                }
                Stabilized = false;
            }
        }

        if (Stabilized) {
            GenerateNextStick();
            GameState = GameStateEnum.Fall;
        }
    }
}

private void CheckDestroyableLine(int X1, int Y1, int DeltaX, int DeltaY) {
    if (Map[X1, Y1] < 0) {
        return;
    }

    int X2 = X1, Y2 = Y1;
    var LineLength = 0;
    while ((X2 >= 0) && (Y2 >= 0) && (X2 < MapWidth) && (Y2 < MapHeight) && (Map[X2, Y2] == Map[X1, Y1])) {
        ++LineLength;
        X2 += DeltaX;
        Y2 += DeltaY;
    }

    if (LineLength >= DestroyableLength) {
        for (var i = 0; i < LineLength; i++) {
            Destroyables.Push(new Vector2(X1 + i * DeltaX, Y1 + i * DeltaY));
        }
    }
}
```

В репозитории помечаем: «Destroying lines of same color».

# Game Over #

Вы, вероятно, заметили забавное поведение игры при достижении верха стакана: вверху начинают мигать разными цветами, сменяя друг друга, бесконечно появляющиеся и тут же застывающие палки. Давайте обработаем ситуацию проигрыша: ничего не будет происходить, пока пользователь не нажмёт клавишу Enter.

Здесь всё просто.

```
protected override void OnUpdateFrame(FrameEventArgs E) {
    // . . .
    } else if (GameStateEnum.Impact == GameState) {
        // . . .

        if (Stabilized) {
            var GameOver = false;
            for (var X = 0; X < MapWidth; X++) {
                if (Map[X, 0] >= 0) {
                    GameOver = true;
                    break;
                }
            }

            if (GameOver) {
                GameState = GameStateEnum.GameOver;
            } else {
                GenerateNextStick();
                GameState = GameStateEnum.Fall;
            }
        }
    }
}

protected void OnKeyDown(object Sender, KeyboardKeyEventArgs E) {
    if (GameStateEnum.Fall == GameState) {
        if ((Key.Left == E.Key) && (StickPosition.X > 0)) {
            --StickPosition.X;
        } else if ((Key.Right == E.Key) && (StickPosition.X + StickLength < MapWidth)) {
            ++StickPosition.X;
        } else if (Key.Up == E.Key) {
            var T = StickColors[0];
            for (var i = 0; i < StickLength - 1; i++) {
                StickColors[i] = StickColors[i + 1];
            }
            StickColors[StickLength - 1] = T;
        }
    } else if (GameStateEnum.GameOver == GameState) {
        if ((Key.Enter == E.Key) || (Key.KeypadEnter == E.Key)) {
            New();
        }
    }
}
```

Коммитим, без лишних размышлений подписав: «Game over».

На этом первая часть разработки закончена. У нас на руках — вполне полнофункциональная игра, в которую уже сейчас можно играть. Во [второй части](Alpha2.md) мы займёмся оформлением. Наложим текстуры, будем показывать текущий счёт, рекордный счёт; палку, которая выпадет следующей. Отцентрируем наконец изображение стакана в окне игры, изобразим его границы. В общем, доведём всё до ума.