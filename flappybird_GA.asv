function flappybird
clear all, clc, close all force
%% Constant Definitions:
GAME_SPEED = 200;
INITIAL_TUBE_X = 150;

% -- Screen parameters
GameVer = '1.0';                    % The first full playable game
GAME.MAX_FRAME_SKIP = [];

GAME.RESOLUTION = [];               % Game Resolution, default at [256 144]
GAME.WINDOW_SCALE = 2;              % The actual size of the window divided by resolution
GAME.FLOOR_TOP_Y = [];              % The y position of upper crust of the floor.
GAME.N_UPDATES_PER_SEC = [];
GAME.FRAME_DURATION = [];           
GAME.GRAVITY = 0.2;              % 0.15; %0.2; %1356;  % empirical gravity constant
      
TUBE.MIN_HEIGHT = [];               % The minimum height of a tube
TUBE.RANGE_HEIGHT = [];             % The range of the height of a tube
TUBE.SUM_HEIGHT = [];               % The summed height of the upper and low tube
TUBE.H_SPACE = [];                  % Horizontal spacing between two tubs
TUBE.V_SPACE = [];                  % Vertical spacing between two tubs
TUBE.WIDTH   = [];                  % The 'actual' width of the detection box
GAMEPLAY.RIGHT_X_FIRST_TUBE = [];   % Xcoord of the right edge of the 1st tube
TubeLayer.Alpha = [];
TubeLayer.CData = [];

% -- Handles
MainFigureHdl = [];
MainAxesHdl = [];
MainCanvasHdl = [];
BirdSpriteHdl = [];
TubeSpriteHdl = [];
FloorSpriteHdl = [];
ScoreInfoBackHdl = [];              % add this line
ScoreInfoForeHdl = [];              % add this line 
ScoreInfoHdl = [];
GameOverHdl = [];
FloorAxesHdl = [];

% -- Game Parameters
MainFigureInitPos = [];
MainFigureSize = [];
MainAxesInitPos = [];               % The initial position of the axes IN the figure
MainAxesSize = [];

InGameParams.CurrentBkg = 1;
InGameParams.CurrentBird = 1;

Flags.IsGameStarted = true;         %
Flags.IsFirstTubeAdded = false;     % Has the first tube been added to TubeLayer
Flags.ResetFloorTexture = true;     % Result the pointer for the floor texture
Flags.NextTubeReady = true;
CloseReq = false;

FlyKeyNames = {'space', 'return', 'uparrow', 'w'};
FlyKeyStatus = false;
FlyKeyValid = true(size(FlyKeyNames));      

% -- Canvases:
MainCanvas = [];
Sprites = [];

% -- Positions:
Bird.COLLIDE_MASK = [];
Bird.INIT_SCREEN_POS = [45 100];        % In [x y] order;
Bird.WorldX = [];
Bird.ScreenPos = [45 100]; %[45 100];   % Center = The 9th element horizontally (1based)
                                        % And the 6th element vertically 
Bird.SpeedXY = [ 0, 0 ];
Bird.Angle = 0;
Bird.XGRID = [];
Bird.YGRID = [];
Bird.CurFrame = 1;
Bird.SpeedY = 0;
Bird.LastHeight = 0;
SinYRange = 44;
SinYPos = [];
SinY = [];

Score = 0;
Best = 0;

Tubes.FrontP = 1;    
Tubes.Actual = 1;
Tubes.ScreenX = [INITIAL_TUBE_X INITIAL_TUBE_X+100 INITIAL_TUBE_X+200]-2;        % The middle of each tube
Tubes.VOffset = ceil(rand(1,3)*105); 

%% -- Game Logic --
% TENTO KOD SA VYKONA IBA PRI INITE HRY, TJ KED HRU ZAPNEM

initVariables();
initWindow();

% ----- moje vlastne premenne ------ %

% NASTAVENIE NEURO-EVOLUCIE
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

NUM_POP = 30;      % pocet jedincov v generacii
H = 1;              % max velkost hladanych parametrov
NUM_GEN = 50;       % pocet generacii   

INPUT_PARAMS = 4; HIDDEN_NEURONS_1 = 4;  
HIDDEN_NEURONS_2 = 2; OUTPUT_PARAMS = 1;      

W1_I = INPUT_PARAMS * HIDDEN_NEURONS_1;
W2_I = W1_I + HIDDEN_NEURONS_1 * HIDDEN_NEURONS_2;
W3_I = W2_I + HIDDEN_NEURONS_2 * OUTPUT_PARAMS;
B1_I = W3_I + HIDDEN_NEURONS_1;
B2_I = B1_I + HIDDEN_NEURONS_2;

ORDER = INPUT_PARAMS * HIDDEN_NEURONS_1 + ...
        HIDDEN_NEURONS_1 * HIDDEN_NEURONS_2 + ...
        HIDDEN_NEURONS_2 * OUTPUT_PARAMS + ...
        HIDDEN_NEURONS_1 + HIDDEN_NEURONS_2;
    
% generovanie novej populacie - pomocne vektory
Space = [ - ones(1,ORDER) * H;
            ones(1,ORDER) * H];
Amp = ones(1,ORDER) * 0.01;

% generovanie novej populacie
vec_pops = genrpop(NUM_POP, Space); 

% vytvorenie vektora pre odkladanie najlepsich jedincov, zvlast pre kazdy ostrov
min_error_across_gens = zeros(NUM_GEN, 1); 
Neuro_params = zeros(NUM_POP, ORDER);

population_index = 1;
generation_index = 1;
fitnes = zeros(NUM_POP, 1);

% ----- moje vlastne premenne ------ %

%% GAMAE LOOP - OUTER, MEANS THAT IT CYCLE OVER EVERY GAME RESTART
while 1
    
    ... TENTO KOD SA VYKONA VZDY IBA RAZ PO RESTARTE HRY, TJ KED PADNEM A DAM HRU ODZNOVA
    initRun;
    stageStartTime = tic;
    c = stageStartTime;  

    % ----- moje vlastne premenne ------ %
    prejdena_vzdialenost=0;
    cakaj_pred_restartom=0;
    CAS_PRED_RESTARTOM=40;
    
    % ----- moje vlastne premenne ------ %
    
    %% GAMAE LOOP - INNER, MEANS THAT IT UPDATES BIRD EVERY FRAME AND COLLECT USER INPUT
    while 1
        
        ...TENTO KOD SA VYKONAVA NEUSTALE, AJ KED JE BIRD PADNUTY
        loops = 0;      
        curTime = toc(stageStartTime);

        % this loop is there because of fps difference in every frame
        % so to be able to have same speed every frame, we need this
        while (curTime >= ((CurrentFrameNo) * GAME.FRAME_DURATION) && loops < GAME.MAX_FRAME_SKIP)
    
            % - tu bude neuro-evolucny algoritmus, ktory bude rozhodovat o skoku.
            ... FlyKeyStatus hovori, ci bol stlaceny medzernik (fly key).
            ... Ak je FlyKeyStatus 1, birdovi sa setne zaporna y-rychlost,
            ... co znamena, ze bird vyskoci.    
            ... Ulohou neuroevolucie bude nastavovat premennu FlyKeyStatus na
            ... zaklade vystupu NN.
            % status = NN.
            nn_out = brain_think(vec_pops, population_index, ...
                [Bird.ScreenPos(2); ...
                 Bird.SpeedY;...
                 Tubes.ScreenX(Tubes.Actual);...
                 Tubes.VOffset(Tubes.Actual)  ]);

            if nn_out > 0
                FlyKeyStatus = true;
            end
   
            % ak nie je gameover, pocitaj vzdialenost, inak pocitaj, kolko
            % treba cakat pred restartom hry
            if ~ gameover
                prejdena_vzdialenost = prejdena_vzdialenost + 1;
            else
                cakaj_pred_restartom = cakaj_pred_restartom + 1;
            end

            % na konci hry vypis hlasku a inkrementuj index jedinca
            if gameover && cakaj_pred_restartom == 1    
                pokuta = 0;
                if fall_to_bottom || fall_to_tube
                    pokuta = 10000000;
                end
                
                fitnes(population_index) = pokuta - prejdena_vzdialenost - 5^Score;
                population_index = population_index + 1;
            end
            
            % chvilku cakaj pred tym ako restartujem hru s novym
            % jedincom/generaciou
            if cakaj_pred_restartom == CAS_PRED_RESTARTOM
                % tu sa vyhodnoti fitnes funkcia pre prave skonceneho jedinca
                FlyKeyStatus = true; 
%                 clc
            end       

            if population_index == NUM_POP + 1
                population_index = 1;                
                min_error_across_gens(generation_index) = min(fitnes);                                            
    
                Best_pop = selbest(vec_pops,fitnes,[2]);
                Work = seltourn(vec_pops,fitnes,NUM_POP-2);
                Work = crossov(Work,2,0);
                Work = mutx(Work,0.15,Space);
                Work = muta(Work,0.17,Amp,Space);
                vec_pops = [Best_pop;Work];
                disp("Generation: " + num2str(generation_index) + " - Fitnes: " + num2str(min_error_across_gens(generation_index)))
                generation_index = generation_index + 1;
            end

            if ~gameover
                if FlyKeyStatus              % If jump key is pressed     
                    Bird.SpeedY = -2.5;
                    FlyKeyStatus = false;
                    Bird.LastHeight = Bird.ScreenPos(2);
                end

                processBird;
                Bird.ScrollX = Bird.ScrollX + 1;
                scrollTubes(1);

                addScore;
                Bird.CurFrame = 3 - floor(double(mod(CurrentFrameNo, 9))/3);
        
                % Update the cycle variables
                if isCollide()
                    gameover = true;
                    fall_to_tube = true;
                elseif Bird.ScreenPos(2) >= 200-5 || Bird.ScreenPos(2) < 0        
                    gameover = true;
                    fall_to_bottom = true;
                end
            end

            CurrentFrameNo = CurrentFrameNo + 1;
            loops = loops + 1;
            frame_updated = true;           
        end
        
        % Redraw the frame if the world has been processed    
        if frame_updated
            set(MainCanvasHdl, 'CData', MainCanvas(1:200,:,:));          
            refreshBird();
            refreshTubes();
            if (~gameover); refreshFloor(CurrentFrameNo); end
            curScoreString = sprintf('%d',(Score));
            set(ScoreInfoForeHdl, 'String', curScoreString);
            set(ScoreInfoBackHdl, 'String', curScoreString);
            drawnow;
            frame_updated = false;
            c = toc(stageStartTime);
        end

        % check, if bird colided and if so, report and save the score
        if fall_to_bottom || fall_to_tube
            if Score > Best
                Best = Score;
                save sprites2.mat Best -append
            end
            score_report = {sprintf('Score: %d', Score), sprintf('Best: %d', Best)};
            set(ScoreInfoHdl, 'Visible','on', 'String', score_report);
            set(ScoreInfoBackHdl, 'Visible','off');
            set(GameOverHdl, 'Visible','on');
            if FlyKeyStatus; break; end
       end
        
        % close handler
        if CloseReq    
            delete(MainFigureHdl);
            clear all;
            return;
        end
    end

    if generation_index == NUM_GEN + 1
        break
    end

end

close all force
figure(1)
plot(min_error_across_gens)
title("Hodnota fitnes funkcie naprieč generáciami")
xlabel("Generácia")
ylabel("Hodnota fitnes [-]")

[~, index] = min(fitnes);
W1 = Neuro_params(index, 1:W1_I)     ; W1 = reshape(W1, HIDDEN_NEURONS_1, INPUT_PARAMS); 
W2 = Neuro_params(index, W1_I+1:W2_I); W2 = reshape(W2, HIDDEN_NEURONS_2, HIDDEN_NEURONS_1);
W3 = Neuro_params(index, W2_I+1:W3_I); W3 = reshape(W3, OUTPUT_PARAMS, HIDDEN_NEURONS_2);
B1 = Neuro_params(index, W3_I+1:B1_I); B1 = reshape(B1, HIDDEN_NEURONS_1, 1);
B2 = Neuro_params(index, B1_I+1:B2_I); B2 = reshape(B2, HIDDEN_NEURONS_2, 1);

BRAIN.W1 = W1;
BRAIN.W2 = W2;
BRAIN.W3 = W3;
BRAIN.B1 = B1;
BRAIN.B2 = B2;

% save best network
disp("Saving best BRAIN to file -BRAIN.mat-...!")
save("BRAIN", "BRAIN")



%% END OF GAME %%
%...
%...
%...
%...
%...
%...
%...
%...
%...
%% FUNCTION SECTION %%

function [output] = brain_think(vec_pops, population_index, X)

    X_norm(1, 1)  = - math_scale_values(X(1), 0, 200, -1, 1);
    X_norm(2, 1)  = - math_scale_values(X(2), -3, 10, -1, 1);
    X_norm(3, 1)  =   math_scale_values(X(3), 20, 150, -1, 1);
    X_norm(4, 1)  =   math_scale_values(X(4) + TUBE.V_SPACE/2, 1 + TUBE.V_SPACE/2, 105 + TUBE.V_SPACE/2, -1, 1);

    W1 = vec_pops(population_index, 1:W1_I);      Neuro_params(population_index, 1:W1_I)      = W1;
    W2 = vec_pops(population_index, W1_I+1:W2_I); Neuro_params(population_index, W1_I+1:W2_I) = W2;
    W3 = vec_pops(population_index, W2_I+1:W3_I); Neuro_params(population_index, W2_I+1:W3_I) = W3;
    B1 = vec_pops(population_index, W3_I+1:B1_I); Neuro_params(population_index, W3_I+1:B1_I) = B1;
    B2 = vec_pops(population_index, B1_I+1:B2_I); Neuro_params(population_index, B1_I+1:B2_I) = B2;
%     disp("--------------------------------------------------")
%     disp("Bird.Y: " + num2str(X(1)) + ", Bird.Dir: " + num2str(X(2)) + ", Pipe.X: " + num2str(X(3)) +", Pipe.Y: " + num2str(X(4)))
%     disp("Bird.Y: " + num2str(X_norm(1)) + ", Bird.Dir: " + num2str(X_norm(2)) + ", Pipe.X: " + num2str(X_norm(3)) +", Pipe.Y: " + num2str(X_norm(4)))
%     disp("--------------------------------------------------")
%     disp("[ Pipe X , Tube Y ]  =  [ " + num2str(X(3) + " , " + num2str(X(4))) + " ]")
    
    W1 = reshape(W1, HIDDEN_NEURONS_1, INPUT_PARAMS);
    W2 = reshape(W2, HIDDEN_NEURONS_2, HIDDEN_NEURONS_1);
    W3 = reshape(W3, OUTPUT_PARAMS, HIDDEN_NEURONS_2);
    B1 = reshape(B1, HIDDEN_NEURONS_1, 1);
    B2 = reshape(B2, HIDDEN_NEURONS_2, 1);

    A1=(W1*X_norm)+B1;      % vstupna/1.skryta vrstva
    O1=tanh(3*A1);
    A2=(W2*O1)+B2;          % 1./2. skryta vrstva
    O2=tanh(3*A2);
    u=W3*O2;
    if u > 0; output = 1; else; output = -1; end
end

function [ newValue ] = math_scale_values( originalValue, minOriginalRange, maxOriginalRange, minNewRange, maxNewRange )
    %   MATH_SCALE_VALUES
    %   Converts a value from one range into another
    %       (maxNewRange - minNewRange)(originalValue - minOriginalRange)
    %    y = ----------------------------------------------------------- + minNewRange      
    %               (maxOriginalRange - minOriginalRange)
    newValue = minNewRange + (((maxNewRange - minNewRange) * (originalValue - minOriginalRange))/(maxOriginalRange - minOriginalRange));

    if newValue > 1; newValue = 1; elseif newValue < -1; newValue = -1; end;
end

function initVariables()
    Sprites = load('sprites2.mat');
    GAME.MAX_FRAME_SKIP = 5;
    GAME.RESOLUTION = [256 144];
    GAME.WINDOW_RES = [256 144];
    GAME.FLOOR_HEIGHT = 56;
    GAME.FLOOR_TOP_Y = GAME.RESOLUTION(1) - GAME.FLOOR_HEIGHT + 1;
    GAME.N_UPDATE_PERSEC = GAME_SPEED; %60;
    GAME.FRAME_DURATION = 1/GAME.N_UPDATE_PERSEC;
    
    TUBE.H_SPACE = 100;           % Horizontal spacing between two tubs
    TUBE.V_SPACE = 64;            % Vertical spacing between two tubs
    TUBE.WIDTH   = 24;            % The 'actual' width of the detection box
    TUBE.MIN_HEIGHT = 36;
    
    TUBE.SUM_HEIGHT = GAME.RESOLUTION(1)-TUBE.V_SPACE-GAME.FLOOR_HEIGHT;
    TUBE.RANGE_HEIGHT = TUBE.SUM_HEIGHT -TUBE.MIN_HEIGHT*2;
    
    TUBE.PASS_POINT = [1 44];
    
    GAMEPLAY.RIGHT_X_FIRST_TUBE = 300;  % Xcoord of the right edge of the 1st tube
    
    % Handles
    MainFigureHdl = [];
    MainAxesHdl = [];
    
    % Game Parameters
    MainFigureInitPos = [500 100];
    MainFigureSize = GAME.WINDOW_RES([2 1]).*2;
    MainAxesInitPos = [0 0]; 
    MainAxesSize = [144 200];

    % Canvases:
    MainCanvas = uint8(zeros([GAME.RESOLUTION 3]));
            
    bird_size = Sprites.Bird.Size;
    [Bird.XGRID, Bird.YGRID] = meshgrid(-ceil(bird_size(2)/2):floor(bird_size(2)/2), ...
        ceil(bird_size(1)/2):-1:-floor(bird_size(1)/2));
    Bird.COLLIDE_MASK = false(12,12);
    [tempx, tempy] = meshgrid(linspace(-1,1,12));
    Bird.COLLIDE_MASK = (tempx.^2 + tempy.^2) <= 1;  
    
    Bird.OSCIL_RANGE = [128 4];
    
    SinY = Bird.OSCIL_RANGE(1) + sin(linspace(0, 2*pi, SinYRange))* Bird.OSCIL_RANGE(2);
    SinYPos = 1;
    Best = Sprites.Best;
end

function initGame()
    TubeLayer.Alpha = false([GAME.RESOLUTION.*[1 2] 3]);
    TubeLayer.CData = uint8(zeros([GAME.RESOLUTION.*[1 2] 3]));

    Bird.Angle = 0;
    Score = 0;
    Flags.ResetFloorTexture = true;
    SinYPos = 1;
    drawToMainCanvas();
    set(MainCanvasHdl, 'CData', MainCanvas);
    set(ScoreInfoHdl, 'Visible','off');
    set(ScoreInfoBackHdl, 'Visible','on');
    set(ScoreInfoForeHdl, 'Visible','off');
    set(GameOverHdl, 'Visible','off');
    set(FloorSpriteHdl, 'CData',Sprites.Floor.CData);
    Tubes.FrontP = 1;  
    Tubes.Actual = 1;
    Tubes.ScreenX = [INITIAL_TUBE_X INITIAL_TUBE_X+100 INITIAL_TUBE_X+200]-2;
    Tubes.VOffset = ceil(rand(1,3)*105);
    refreshTubes;
    for i = 1:3
        set(TubeSpriteHdl(i),'CData',Sprites.TubGap.CData,...
            'AlphaData',Sprites.TubGap.Alpha);
        redrawTube(i);
    end
end

function initRun()
    initGame();
    CurrentFrameNo = double(0);
    fall_to_bottom = false;
    fall_to_tube = false;
    gameover = false;
    
    FlyKeyStatus=true;

    Bird.ScreenPos(2) = SinY(SinYPos);
    SinYPos = mod(SinYPos, SinYRange)+1;
    Bird.ScrollX = 0;

    rng default
end

function initWindow()
    MainFigureHdl = figure('Name', ['Flappy Bird ' GameVer], ...
        'NumberTitle' ,'off', ...
        'Units', 'pixels', ...
        'Position', [MainFigureInitPos, MainFigureSize], ...
        'MenuBar', 'figure', ...
        'Renderer', 'OpenGL',...
        'Color',[0 0 0], ...
        'KeyPressFcn', @stl_KeyPressFcn, ...
        'WindowKeyPressFcn', @stl_KeyDown,...
        'WindowKeyReleaseFcn', @stl_KeyUp,...
        'CloseRequestFcn', @stl_CloseReqFcn);
    FloorAxesHdl = axes('Parent', MainFigureHdl, ...
        'Units', 'normalized',...
        'Position', [MainAxesInitPos, (1-MainAxesInitPos.*2) .* [1 56/256]], ...
        'color', [1 1 1], ...
        'XLim', [0 MainAxesSize(1)]-0.5, ...
        'YLim', [0 56]-0.5, ...
        'YDir', 'reverse', ...
        'NextPlot', 'add', ...
        'Visible', 'on',...
        'XTick',[], 'YTick', []);
    MainAxesHdl = axes('Parent', MainFigureHdl, ...
        'Units', 'normalized',...
        'Position', [MainAxesInitPos + [0 (1-MainAxesInitPos(2).*2)*56/256], (1-MainAxesInitPos.*2).*[1 200/256]], ...
        'color', [1 1 1], ...
        'XLim', [0 MainAxesSize(1)]-0.5, ...
        'YLim', [0 MainAxesSize(2)]-0.5, ...
        'YDir', 'reverse', ...
        'NextPlot', 'add', ...
        'Visible', 'on', ...
        'XTick',[], ...
        'YTick',[]);    
    
    MainCanvasHdl = image([0 MainAxesSize(1)-1], [0 MainAxesSize(2)-1], [],...
        'Parent', MainAxesHdl,...
        'Visible', 'on');
    TubeSpriteHdl = zeros(1,3);
    for i = 1:3
        TubeSpriteHdl(i) = image([0 26-1], [0 304-1], [],...
        'Parent', MainAxesHdl,...
        'Visible', 'on');
    end
    
    BirdSpriteHdl = surface(Bird.XGRID-100,Bird.YGRID-100, ...
        zeros(size(Bird.XGRID)), Sprites.Bird.CDataNan(:,:,:,1), ...
        'CDataMapping', 'direct',...
        'EdgeColor','none', ...
        'Visible','on', ...
        'Parent', MainAxesHdl);
    FloorSpriteHdl = image(0, 0,[],...
        'Parent', FloorAxesHdl, ...
        'Visible', 'on ');
   
    ScoreInfoBackHdl = text(72, 50, '0', ...
        'FontName', 'Helvetica', 'FontSize', 30, 'HorizontalAlignment', 'center','Color',[0,0,0], 'Visible','off');
    ScoreInfoForeHdl = text(70.5, 48.5, '0', ...
        'FontName', 'Helvetica', 'FontSize', 30, 'HorizontalAlignment', 'center', 'Color',[1 1 1], 'Visible','off');
    GameOverHdl = text(72, 70, 'GAME OVER', ...
        'FontName', 'Arial', 'FontSize', 20, 'HorizontalAlignment', 'center','Color',[1 0 0], 'Visible','off');
    
    ScoreInfoHdl = text(72, 110, 'Best', ...
        'FontName', 'Helvetica', 'FontSize', 20, 'FontWeight', 'Bold', 'HorizontalAlignment', 'center','Color',[1 1 1], 'Visible', 'off');
end

function processBird()
    Bird.ScreenPos(2) = Bird.ScreenPos(2) + Bird.SpeedY;
    Bird.SpeedY = Bird.SpeedY + GAME.GRAVITY;
    if Bird.SpeedY < 0
        Bird.Angle = max(Bird.Angle - pi/10, -pi/10);
    else
        if Bird.ScreenPos(2) < Bird.LastHeight
            Bird.Angle = -pi/10;
        else
            Bird.Angle = min(Bird.Angle + pi/30, pi/2);
        end
    end
end

function drawToMainCanvas()    
    % Redraw the background
    MainCanvas = Sprites.Bkg.CData(:,:,:,InGameParams.CurrentBkg);
    
    TubeFirstCData = TubeLayer.CData(:, 1:GAME.RESOLUTION(2), :);
    TubeFirstAlpha = TubeLayer.Alpha(:, 1:GAME.RESOLUTION(2), :);
    MainCanvas(TubeFirstAlpha) = TubeFirstCData (TubeFirstAlpha);
end

function scrollTubes(offset)
    Tubes.ScreenX = Tubes.ScreenX - offset;
    if Tubes.ScreenX(Tubes.FrontP) <=-26
        Tubes.ScreenX(Tubes.FrontP) = Tubes.ScreenX(Tubes.FrontP) + 300;
        Tubes.VOffset(Tubes.FrontP) = ceil(rand*105);
        redrawTube(Tubes.FrontP);
        Tubes.FrontP = mod((Tubes.FrontP),3)+1;
        Flags.NextTubeReady = true;
    end

    % actualize which tube is ahead of bird
    if Tubes.ScreenX(Tubes.FrontP) == 20
        Tubes.Actual = mod((Tubes.Actual),3)+1;
    end
end

function refreshTubes()
    % Refreshing Scheme 1: draw the entire tubes but only shows a part
    % of each
    for i = 1:3
        set(TubeSpriteHdl(i), 'XData', Tubes.ScreenX(i) + [0 26-1]);
    end
end

function refreshFloor(frameNo)
    offset = mod(frameNo, 24);
    set(FloorSpriteHdl, 'XData', -offset);
end

function redrawTube(i)
    set(TubeSpriteHdl(i), 'YData', -(Tubes.VOffset(i)-1));
end


%% --- Math Functions for handling Collision / Rotation etc. ---
function collide_flag = isCollide()
    collide_flag = 0;
    if Bird.ScreenPos(1) >= Tubes.ScreenX(Tubes.FrontP)-5 && ...
            Bird.ScreenPos(1) <= Tubes.ScreenX(Tubes.FrontP)+6+25    
    else
        return;
    end
    
    GapY = [128 177] - (Tubes.VOffset(Tubes.FrontP)-1);
    
    if Bird.ScreenPos(2) < GapY(1)+4 || Bird.ScreenPos(2) > GapY(2)-4
        collide_flag = 1;
    end
    return;
end

function addScore()
    if Tubes.ScreenX(Tubes.FrontP) < 40 && Flags.NextTubeReady
        Flags.NextTubeReady = false;
        Score = Score + 1;
    end
end

function refreshBird()
    % move bird to pos [X Y],
    % and rotate the bird surface by X degrees, anticlockwise = +
    cosa = cos(Bird.Angle);
    sina = sin(Bird.Angle);
    xrotgrid = cosa .* Bird.XGRID + sina .* Bird.YGRID;
    yrotgrid = sina .* Bird.XGRID - cosa .* Bird.YGRID;
    xtransgrid = xrotgrid + Bird.ScreenPos(1)-0.5;
    ytransgrid = yrotgrid + Bird.ScreenPos(2)-0.5;
    set(BirdSpriteHdl, 'XData', xtransgrid, ...
        'YData', ytransgrid, ...
        'CData', Sprites.Bird.CDataNan(:,:,:, Bird.CurFrame));
end    
        
%% -- Callbacks --

function stl_KeyUp(hObject, ~, ~)
    key = get(hObject,'CurrentKey');
    % Remark the released keys as valid
    FlyKeyValid = FlyKeyValid | strcmp(key, FlyKeyNames);
end

function stl_KeyDown(hObject, ~, ~)
    key = get(hObject,'CurrentKey');
    
    % Has to be both 'pressed' and 'valid';
    % Two key presses at the same time will be counted as 1 key press
    down_keys = strcmp(key, FlyKeyNames);
    FlyKeyStatus = any(FlyKeyValid & down_keys);
    FlyKeyValid = FlyKeyValid & (~down_keys);
end

function stl_KeyPressFcn(hObject, ~, ~)
    curKey = get(hObject, 'CurrentKey');
    switch true
        case strcmp(curKey, 'escape') 
            CloseReq = true;            
    end
end

function stl_CloseReqFcn(~, ~, ~)
    CloseReq = true;
end

end