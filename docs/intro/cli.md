import { useState, useEffect } from 'react';
import { UserProfile } from '../App';
import { BuildingData } from './GameWorld';
import { Card } from './ui/card';
import { Button } from './ui/button';
import { ArrowLeft, TrendingDown, TrendingUp, AlertTriangle, DollarSign, Zap } from 'lucide-react';

interface VisualChallengeProps {
  userProfile: UserProfile;
  building: BuildingData;
  onComplete: (success: boolean, reward: number) => void;
  onCancel: () => void;
}

interface StockData {
  company: string;
  currentPrice: number;
  change: number;
  shares: number;
}

export function VisualChallenge({ userProfile, building, onComplete, onCancel }: VisualChallengeProps) {
  const [phase, setPhase] = useState<'intro' | 'action' | 'result'>('intro');
  const [selectedAction, setSelectedAction] = useState<string | null>(null);
  const [stocks, setStocks] = useState<StockData[]>([]);
  const [timer, setTimer] = useState(30);

  // Generate stock data based on user interests
  useEffect(() => {
    const interest = userProfile.interests[0] || 'games';
    const companies: Record<string, string[]> = {
      anime: ['Клан Учиха Corp', 'Наруто Studios', 'Акацуки Industries'],
      games: ['PixelCraft Games', 'CyberSport Ltd', 'GameDev Studios'],
      movies: ['Hollywood Dreams', 'Cinema Magic', 'Film Production Inc'],
      music: ['SoundWave Records', 'Music Masters', 'Beat Factory'],
      books: ['Publisher House', 'Book World', 'Reading Empire'],
      art: ['Art Gallery Corp', 'Creative Studios', 'Design Masters']
    };

    const companyList = companies[interest] || companies.games;
    
    const newStocks: StockData[] = companyList.map(company => ({
      company,
      currentPrice: Math.floor(Math.random() * 500) + 100,
      change: (Math.random() * 60) - 30, // -30% to +30%
      shares: Math.floor(Math.random() * 100) + 50
    }));

    setStocks(newStocks);
  }, [userProfile.interests]);

  // Timer countdown
  useEffect(() => {
    if (phase === 'action' && timer > 0) {
      const interval = setInterval(() => {
        setTimer(prev => prev - 1);
      }, 1000);
      return () => clearInterval(interval);
    } else if (timer === 0 && phase === 'action') {
      handleSubmit();
    }
  }, [phase, timer]);

  const handleStartChallenge = () => {
    setPhase('action');
  };

  const handleSubmit = () => {
    setPhase('result');
  };

  const handleFinish = () => {
    const success = selectedAction !== null;
    const reward = success ? 500 : 100;
    onComplete(success, reward);
  };

  const getScenarioText = () => {
    const interest = userProfile.interests[0] || 'games';
    const scenarios: Record<string, string> = {
      anime: 'Акции аниме-студий резко падают из-за скандала! Новый проект провалился. Что делать?',
      games: 'Игровая компания теряет деньги после провала крупного релиза. Рынок в панике!',
      movies: 'Киностудия рискует на мега-бюджетном блокбастере. Инвесторы нервничают!',
      music: 'Музыкальный лейбл в кризисе - главный артист ушел к конкуренту!',
      books: 'Издательство терпит убытки. Продажи книг резко упали!',
      art: 'Художественная галерея на грани банкротства из-за новых конкурентов!'
    };
    return scenarios[interest] || scenarios.games;
  };

  return (
    <div className="relative w-full h-full bg-gradient-to-br from-purple-900 via-indigo-900 to-blue-900 overflow-hidden">
      {/* Animated Background */}
      <div className="absolute inset-0">
        {Array.from({ length: 50 }).map((_, i) => (
          <div
            key={i}
            className="absolute w-2 h-2 bg-purple-400/30 rounded-full animate-pulse"
            style={{
              left: `${Math.random() * 100}%`,
              top: `${Math.random() * 100}%`,
              animationDelay: `${Math.random() * 3}s`,
              animationDuration: `${2 + Math.random() * 3}s`
            }}
          />
        ))}
      </div>

      {/* Back Button */}
      <Button
        onClick={onCancel}
        className="absolute top-6 left-6 z-50 bg-black/50 hover:bg-black/70 text-white border border-white/30"
        variant="ghost"
      >
        <ArrowLeft className="w-5 h-5 mr-2" />
        Вернуться в мир
      </Button>

      {/* Building Interior */}
      <div className="relative z-10 h-full flex items-center justify-center p-8">
        <div className="max-w-6xl w-full">
          {/* Intro Phase */}
          {phase === 'intro' && (
            <div className="text-center animate-fade-in">
              <div className="text-9xl mb-8 animate-bounce">
                {building.icon}
              </div>
              <h1 className="text-5xl text-white mb-4">{building.name}</h1>
              <p className="text-2xl text-purple-200 mb-8">
                Добро пожаловать! Готов к испытанию?
              </p>
              
              <Card className="bg-gradient-to-r from-red-500/30 to-orange-500/30 border-2 border-red-500 p-8 mb-8 max-w-3xl mx-auto">
                <AlertTriangle className="w-16 h-16 text-red-400 mx-auto mb-4 animate-pulse" />
                <h2 className="text-3xl text-white mb-4">СИТУАЦИЯ</h2>
                <p className="text-xl text-white">
                  {getScenarioText()}
                </p>
              </Card>

              <Button
                onClick={handleStartChallenge}
                size="lg"
                className="bg-gradient-to-r from-yellow-400 to-orange-500 hover:from-yellow-500 hover:to-orange-600 text-white px-12 py-6 text-xl"
              >
                Начать испытание
              </Button>
            </div>
          )}

          {/* Action Phase */}
          {phase === 'action' && (
            <div className="animate-fade-in">
              {/* Timer */}
              <div className="text-center mb-6">
                <Card className="inline-block bg-black/60 backdrop-blur-lg border-2 border-yellow-400 p-4">
                  <div className="flex items-center gap-3">
                    <Zap className="w-6 h-6 text-yellow-400 animate-pulse" />
                    <span className="text-3xl text-white">⏱️ {timer}s</span>
                  </div>
                </Card>
              </div>

              <h2 className="text-3xl text-white text-center mb-8">
                Выбери компанию для инвестиции
              </h2>

              {/* Stock Market Display */}
              <div className="grid md:grid-cols-3 gap-6 mb-8">
                {stocks.map((stock, index) => {
                  const isSelected = selectedAction === stock.company;
                  const isPositive = stock.change > 0;

                  return (
                    <Card
                      key={index}
                      onClick={() => setSelectedAction(stock.company)}
                      className={`
                        cursor-pointer transition-all duration-300 transform hover:scale-105
                        ${isSelected 
                          ? 'bg-gradient-to-br from-green-600 to-emerald-600 border-4 border-yellow-400 scale-105' 
                          : 'bg-black/60 backdrop-blur-lg border-2 border-purple-500/50 hover:border-purple-400'
                        }
                        p-6
                      `}
                    >
                      {/* Company Name */}
                      <h3 className="text-xl text-white mb-4">{stock.company}</h3>

                      {/* Stock Chart Simulation */}
                      <div className="bg-black/40 rounded-lg p-4 mb-4 h-32 relative overflow-hidden">
                        <svg className="w-full h-full">
                          <path
                            d={`M 0 ${isPositive ? 80 : 20} Q 25 ${isPositive ? 60 : 40} 50 ${isPositive ? 40 : 60} T 100 ${isPositive ? 20 : 80}`}
                            stroke={isPositive ? '#10b981' : '#ef4444'}
                            strokeWidth="3"
                            fill="none"
                            className="animate-pulse"
                          />
                        </svg>
                        <div className="absolute top-2 right-2 flex items-center gap-1">
                          {isPositive ? (
                            <TrendingUp className="w-5 h-5 text-green-400" />
                          ) : (
                            <TrendingDown className="w-5 h-5 text-red-400" />
                          )}
                          <span className={isPositive ? 'text-green-400' : 'text-red-400'}>
                            {isPositive ? '+' : ''}{stock.change.toFixed(1)}%
                          </span>
                        </div>
                      </div>

                      {/* Stock Info */}
                      <div className="space-y-2">
                        <div className="flex justify-between text-purple-200">
                          <span>Цена:</span>
                          <span className="text-white">{stock.currentPrice}₸</span>
                        </div>
                        <div className="flex justify-between text-purple-200">
                          <span>Акций:</span>
                          <span className="text-white">{stock.shares}</span>
                        </div>
                        <div className="flex justify-between text-purple-200">
                          <span>Стоимость:</span>
                          <span className="text-yellow-400">
                            {(stock.currentPrice * stock.shares).toLocaleString()}₸
                          </span>
                        </div>
                      </div>

                      {isSelected && (
                        <div className="mt-4 bg-yellow-400 text-purple-900 px-4 py-2 rounded-lg text-center">
                          ✓ Выбрано
                        </div>
                      )}
                    </Card>
                  );
                })}
              </div>

              <div className="text-center">
                <Button
                  onClick={handleSubmit}
                  disabled={!selectedAction}
                  size="lg"
                  className="bg-gradient-to-r from-green-500 to-emerald-600 hover:from-green-600 hover:to-emerald-700 text-white px-12 py-6"
                >
                  Подтвердить инвестицию
                </Button>
              </div>
            </div>
          )}

          {/* Result Phase */}
          {phase === 'result' && (
            <div className="text-center animate-fade-in">
              <div className="text-9xl mb-8 animate-bounce">
                {selectedAction ? '🎉' : '😅'}
              </div>
              
              <h2 className="text-5xl text-white mb-4">
                {selectedAction ? 'Поздравляем!' : 'Не страшно!'}
              </h2>
              
              <p className="text-2xl text-purple-200 mb-8">
                {selectedAction 
                  ? 'Ты сделал инвестицию! Компания восстанавливается.' 
                  : 'В следующий раз будь быстрее!'}
              </p>

              <Card className="bg-gradient-to-r from-yellow-500/30 to-orange-500/30 border-2 border-yellow-400 p-8 mb-8 max-w-2xl mx-auto">
                <DollarSign className="w-16 h-16 text-yellow-400 mx-auto mb-4" />
                <h3 className="text-3xl text-white mb-2">Награда</h3>
                <p className="text-5xl text-yellow-400">
                  +{selectedAction ? '500' : '100'} 💰
                </p>
                <p className="text-purple-200 mt-4">
                  {selectedAction 
                    ? 'За успешное решение финансовой задачи!' 
                    : 'За попытку! Продолжай учиться!'}
                </p>
              </Card>

              <Button
                onClick={handleFinish}
                size="lg"
                className="bg-gradient-to-r from-purple-500 to-pink-600 hover:from-purple-600 hover:to-pink-700 text-white px-12 py-6"
              >
                Вернуться в мир
              </Button>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

## Install the Ionic CLI

Before proceeding, make sure your computer has [Node.js](../reference/glossary.md#node) installed. See [these instructions](environment.md) to set up an environment for Ionic.

Install the Ionic CLI with npm:

```shell
npm install -g @ionic/cli
```

If there was a previous installation of the Ionic CLI, it will need to be uninstalled due to a change in package name.

```shell
$ npm uninstall -g ionic
$ npm install -g @ionic/cli

```

:::note
The `-g` option means _install globally_. When packages are installed globally, `EACCES` permission errors can occur.
Consider setting up npm to operate globally without elevated permissions. See [Resolving Permission Errors](../developing/tips.md#resolving-permission-errors) for more information.
:::

## Start an App

Create an Ionic app using one of the pre-made app templates, or a blank one to start fresh. The three most common starters are the `blank` starter, `tabs` starter, and `sidemenu` starter. Get started with the `ionic start` command:

```shell
ionic start
```

![Three thumbnail previews of Ionic app templates: blank, tabs, and side menu.](/img/installation/start-app-thumbnails.png 'Ionic App Starter Templates')

To learn more about starting Ionic apps, see the [Starting Guide](../developing/starting.md).

## Run the App

The majority of Ionic app development can be spent right in the browser using the `ionic serve` command:

```shell
$ cd myApp
$ ionic serve
```

There are a number of other ways to run an app, it's recommended to start with this workflow. To develop and test apps on devices and emulators, see the [Running an App Guide](../developing/previewing.md).
