# Spectrogram
Real-Time Spectrogram of Audio Signal
--------------------------------------------------------------------------------------------------------------------------------
function color_range(hobject,event,spec_axes)


range = get(hobject,'String');
range_num = str2num(range);
caxis(spec_axes,[0 range_num]); 

end
--------------------------------------------------------------------------------------------------------------------------------

function freq_range(hobject,event,freq_text,spectrogram,...
    spec_axes)

% Change the y-axis in the GUI
freq_Range = get(hobject,'value');
set(spec_axes,'ylim',[0 freq_Range]);
drawnow;

% GUI display 
frange = get(hobject,'value');
frange = floor(frange);
frange_str = num2str(frange);
frange_display = strcat('Frequency:',frange_str,'(Hz)');
set(freq_text,'string',frange_display);

end
--------------------------------------------------------------------------------------------------------------------------------
clear
clc
%create a figure to house the GUI
figure


%-----------------------------------------------------------------
%---------------------------Plot Axes---------------------------
%-----------------------------------------------------------------


% spectrogram image
spec_axes = axes('Units','normalized','Position',[.05,.075,.7,.9],...
    'Visible','off');
C = zeros(256,800); %CData for imagesc
X = [0 2];
Y = [0 4000];
spectrogram = imagesc(X,Y,C); 
colorbar;
axis xy  % make the axis increase from zero
xlabel (spec_axes, 'Time(s)');
ylabel (spec_axes, 'Frequency(Hz)');

%-----------------------------------------------------------------
%-----------------------------------------------------------------

%-----------------------------------------------------------------
%---------------------------Color Range-----------------------
%-----------------------------------------------------------------


color_text = uicontrol('Style','text','String','Enter Color Range:',...
            'units','normalized',...
            'position',[0.8    0.75    0.1    0.03]);

color_box = uicontrol('Style','edit','units','normalized',...
            'position',[0.8    0.7    0.08    0.04],...
            'Callback',{@color_range,spec_axes});


%-----------------------------------------------------------------
%-----------------------------------------------------------------

%-----------------------------------------------------------------
%---------------------------Frequency Range-----------------------
%-----------------------------------------------------------------

% frequncy text display
freq_text = uicontrol('Style','text','String','Frequency:4000(Hz)',...
            'units','normalized',...
            'position',[0.8    0.05    0.1    0.03]);


% The frequency range slider
freq_slid = uicontrol('Style','slider',...
           'Min',100,'Max',4000,'Value',4000,...
            'SliderStep',([1, 1] / (4000 - 100)),...
           'units','normalized','Position',[.8,.04,.25,.05],...
           'Callback',{@freq_range,freq_text,spectrogram,...
    spec_axes});
       
%-----------------------------------------------------------------
%-----------------------------------------------------------------

%-----------------------------------------------------------------
%---------------------------Sample Rate---------------------------
%-----------------------------------------------------------------
% The sample rate menu
sample_rate = uicontrol('style','popupmenu',...
           'String',{'8000Hz','11025Hz','22050Hz',...
           '44100Hz','48000Hz'},...
            'units','normalized',...
            'position',[0.72    0.77    0.08    0.1],...
            'Callback',{@sample_size,freq_slid,freq_text,spectrogram,...
    spec_axes});
%smaple rate title
srate_text = uicontrol('Style','text','String','Sample Rate',...
            'units','normalized',...
            'position',[0.72    0.88    0.1    0.03]);
%-----------------------------------------------------------------
%-----------------------------------------------------------------
        
%-----------------------------------------------------------------
%---------------------------Sample Range---------------------------
%-----------------------------------------------------------------

% time range span slider title, display
srange_text = uicontrol('Style','text','String','Time Range:2(s)',...
            'units','normalized',...
            'position',[0.8    0.71    0.1    0.03]);

% The time range slider
sample_slid = uicontrol('style','Slider',...        
            'Min',2,'Max',20,'Value',2,...
            'SliderStep',([1, 1] / (20 - 2)),...
            'units','normalized',...
            'position',[0.8  0.6 .25 .05],...
            'Callback',{@time_range,srange_text,spectrogram,...
            spec_axes});

        
%-----------------------------------------------------------------
%-----------------------------------------------------------------


%-----------------------------------------------------------------
%---------------------------Window Size-----------------------------
%-----------------------------------------------------------------

% Window size display
window_text = uicontrol('Style','text','String','Window Size:10(ms)',...
            'units','normalized',...
            'position',[0.8    0.49    0.1    0.03]);
        
% The window size slider
window_slid = uicontrol('style','Slider',...        
            'Min',10,'Max',500,'Value',10,...
            'SliderStep',([1, 1] / (500 - 10)),...
            'units','normalized',...
            'position',[0.8 0.38 .25 .05],...
            'Callback',{@window_size, window_text});
 


%-----------------------------------------------------------------
%-----------------------------------------------------------------





%-----------------------------------------------------------------
%---------------------------Record button panel-------------------
%-----------------------------------------------------------------

record_panel = uipanel('Title','Record','units','normalized',...
               'Position',[0.85    0.82    0.1    0.15]);
           
stop_button = uicontrol('Parent',record_panel, 'String',...
    'Stop','units','normalized','Position',[0.18 0.1 .7 .3], ...
    'Callback', @stop_fun);
          
                      
start_button = uicontrol('Parent',record_panel, 'String',...
    'Start','units','normalized','Position',[0.18 0.6 .7 .3], ...
    'Callback',{@start_fun,sample_rate, sample_slid,window_slid,...
    spectrogram,freq_slid});

%-----------------------------------------------------------------
%-----------------------------------------------------------------

% Align the objects
align([sample_rate sample_slid freq_slid window_slid ...
    color_box],'Left','Distribute');
align([sample_slid srange_text],'Left','Fixed',5);
align([window_slid window_text],'Left','Fixed',5);
align([freq_slid freq_text],'Left','Fixed',5);              
align([color_box color_text],'Left','Fixed',5); 

--------------------------------------------------------------------------------------------------------------------------------
function sample_size(hobject,event,freq_slid,freq_text,spectrogram,...
    spec_axes)

% initialize the frequency slider when a new sample rate is set
set(freq_slid,'Max',10000);
set(freq_slid,'SliderStep',([1, 1] / (10000 - 100)));


% find the sample rate
rate = get(hobject, 'String');
rate_val = get(hobject,'Value');
str_entered = rate(rate_val);
if strcmp(str_entered, '8000Hz');
    fs = 8000;
elseif strcmp(str_entered,'11025Hz');
    fs = 11025;
elseif strcmp(str_entered,'22050Hz');
    fs = 22050;
elseif strcmp(str_entered,'44100Hz');
    fs = 44100;
else strcmp(str_entered,'48000Hz');
    fs = 48000;
end

% change the frequency slider limit & display
freq = fs/2;
max = get(freq_slid,'Max');

if max>freq
    set(freq_slid,'Max',freq);
    set(freq_slid,'SliderStep',([1, 1] / (freq - 100)));
    set(freq_slid,'Value',freq);
    set(spec_axes,'ylim',[0 freq]);
    frange_display = strcat('Frequency:',num2str(freq),'(Hz)');
    set(freq_text,'String',frange_display);
    set(spectrogram,'YData',[0 freq]);
else
    set(freq_slid,'Value',10000);
    set(spec_axes,'ylim',[0 10000]);
    set(freq_text,'String','Frequency:10000(Hz)');
    set(spectrogram,'YData',[0 10000]);
end
drawnow;



end

--------------------------------------------------------------------------------------------------------------------------------
function start_fun(hobject,event, sample_rate, sample_slid,window_slid,...
    spectrogram,freq_slid)

global stop 
stop = 0;

% find the time range
range = get(sample_slid,'value');
range = range*1000; %convert to milliseconds

% find the window size
window = get(window_slid,'value'); % in milliseconds

% find the sample rate
rate = get(sample_rate, 'String');
rate_val = get(sample_rate,'Value');
str_entered = rate(rate_val);
if strcmp(str_entered, '8000Hz');
    fs = 8000;
elseif strcmp(str_entered,'11025Hz');
    fs = 11025;
elseif strcmp(str_entered,'22050Hz');
    fs = 22050;
elseif strcmp(str_entered,'44100Hz');
    fs = 44100;
else strcmp(str_entered,'48000Hz');
    fs = 48000;
end

% find the frequency range
max_freq = get(freq_slid,'Max');  % maximum frequency 
select_freq = get(freq_slid,'Value');  % selected frequency


% calculate the NFFT required to meet 256 bins requirement------------
window_size = floor((window/1000)*fs); %find the number of samples in a window
NFFT = 2^nextpow2(window_size)*4; % Next power of 2 from length of window
fraction_freq = select_freq/max_freq; % fraction of the frequency viewed
minimum_bins = fraction_freq*NFFT; % minimum number of bins displayed
while minimum_bins < 512
    NFFT = 2^nextpow2(NFFT+1);
    minimum_bins = fraction_freq*NFFT;
end
%-----------------------------------------------------------------------

% initializing the hop size and matrices-------------------------
hop_size = 0.05*fs; % 50ms hop size(in samples)
hop_milli = 50; %in milliseconds
num_rows = NFFT/2+1;  % half the length of the fft function
image_col = round(range/hop_milli);  % display length divided by hop size
image_data = zeros(num_rows,image_col);  %initialize the main data matrix
spectra_data = zeros(num_rows,0);  %initialize the window estimates
%-----------------------------------------------------------------------

c = 1; % initialize the window index starting point
d = window_size; % initialize the window index ending point

% start recording
recObj = audiorecorder(fs,8,1);
record(recObj);
on = isrecording(recObj);
if on==0
        disp('Not recording');
else
        disp('Recording');
end
pause(.5);

while true
tic
myRecording = getaudiodata(recObj); % get the audio data
a = length(myRecording);
 
    while d<a          % do the sliding window calculations
    k = myRecording(c:d);
    w = blackman(window_size,'periodic');
    Y = k.*w; % windowing
    X = fft(Y,NFFT); 
    spectra_data = [spectra_data abs(X(1:NFFT/2+1))];
    d = d+hop_size;
    c = c+hop_size;
    end
   
[num_rows, num_col] = size(spectra_data);
image_data = [image_data(:,(num_col+1):end) spectra_data];    
set(spectrogram,'CData',image_data);
drawnow;
spectra_data = zeros(num_rows,0);
toc     
%stop the recording
if stop == 1
    disp('Stopped Recording');
    stop = 0;
    break;
end

end


end   

--------------------------------------------------------------------------------------------------------------------------------
function stop_fun(hobject, event, handles)

global stop
stop = 1;


end

--------------------------------------------------------------------------------------------------------------------------------
function time_range(hobject, event, srange_text,spectrogram,...
    spec_axes)

% Change the x-axis in the GUI
Time_Range = get(hobject,'value');
set(spec_axes,'xlim',[0 Time_Range]);
set(spectrogram,'XData',[0 Time_Range]);
drawnow;

% change the displayed label to reflect the value selected
Time_Range = floor(Time_Range);
Time_str = num2str(Time_Range);
Time_display = strcat('Time Range:',Time_str,'(s)');
set(srange_text,'string',Time_display);





end

--------------------------------------------------------------------------------------------------------------------------------
function window_size(hobject, event, window_text)


window = get(hobject,'value');
window = floor(window);
window_str = num2str(window);

window_display = strcat('Window Size:',window_str,'(ms)');

set(window_text,'string',window_display);





end

--------------------------------------------------------------------------------------------------------------------------------
