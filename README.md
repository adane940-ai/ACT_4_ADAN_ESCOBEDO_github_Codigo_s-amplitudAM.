# ACT_4_ADAN_ESCOBEDO_github_Codigo_s-amplitudAM.
Codigo en MATLAB codigo creacion de ondas de amplitud modulada AM
clc; clear; close all;

fprintf('==============================================\n');
fprintf('  Sistema de Modulación en Amplitud (AM)\n');
fprintf('==============================================\n\n');

% -------------------------------------------------------------------------
%  SECCIÓN 1: PARÁMETROS GLOBALES DE SIMULACIÓN
% -------------------------------------------------------------------------

fs      = 100e3;      % Frecuencia de muestreo [Hz] (100 kHz)
T       = 0.05;       % Duración total de la señal [s]
t       = 0:1/fs:T-1/fs;  % Vector de tiempo

% -- Señal del mensaje (frecuencia fa Am) --
fm      = 1000;       % Frecuencia de la señal mensaje [Hz] (1 kHz)
Am      = 1.0;        % Amplitud del mensaje [V]

% -- Señal portadora --
fc      = 20e3;       % Frecuencia de la portadora [Hz] (20 kHz)
Ac      = 2.0;        % Amplitud de la portadora [V]

% -- Índice de modulación --
mu      = Am / Ac;    % Índice de modulación (debe ser 0 < mu <= 1)

fprintf('[INFO] Parámetros:\n');
fprintf('  Frecuencia mensaje   : %g Hz\n', fm);
fprintf('  Frecuencia portadora : %g Hz\n', fc);
fprintf('  Amplitud mensaje     : %.2f V\n', Am);
fprintf('  Amplitud portadora   : %.2f V\n', Ac);
fprintf('  Índice de modulación : %.2f\n\n', mu);

% -------------------------------------------------------------------------
%  SECCIÓN 2: GENERACIÓN DE SEÑALES BASE
% -------------------------------------------------------------------------

% Señal del mensaje: tono sinusoidal simple
m_t = Am * cos(2*pi*fm*t);

% Señal portadora de alta frecuencia
c_t = Ac * cos(2*pi*fc*t);

% -------------------------------------------------------------------------
%  SECCIÓN 3: MODULACIÓN EN AMPLITUD (AM)
%
%  Fórmula: s(t) = Ac * [1 + mu * m_n(t)] * cos(2*pi*fc*t)
%  donde m_n(t) es el mensaje normalizado a rango [-1, 1]
% -------------------------------------------------------------------------

m_n   = m_t / Am;                        % Mensaje normalizado
s_t   = Ac * (1 + mu * m_n) .* cos(2*pi*fc*t);  % Señal AM modulada

% -------------------------------------------------------------------------
%  SECCIÓN 4: ANÁLISIS EN FRECUENCIA (FFT)
% -------------------------------------------------------------------------

N       = length(t);
f_axis  = (-N/2:N/2-1) * (fs/N);        % Eje de frecuencia centrado

% FFT de cada señal (desplazada al centro)
M_f  = fftshift(abs(fft(m_t,  N) / N));
C_f  = fftshift(abs(fft(c_t,  N) / N));
S_f  = fftshift(abs(fft(s_t,  N) / N));

% =========================================================================
%  FIGURA 1: Señal del mensaje y portadora
% =========================================================================
figure('Name','Señales Base','NumberTitle','off','Color','w','Position',[50 500 1200 450]);

subplot(2,2,1);
plot(t*1e3, m_t, 'b', 'LineWidth',1.4);
xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
title('Señal del Mensaje m(t) — Dominio del Tiempo');
xlim([0 5]); grid on;

subplot(2,2,2);
plot(f_axis/1e3, M_f, 'b', 'LineWidth',1.4);
xlabel('Frecuencia [kHz]'); ylabel('|M(f)|');
title('Espectro del Mensaje — Dominio de la Frecuencia');
xlim([-5 5]); grid on;

subplot(2,2,3);
plot(t*1e3, c_t, 'r', 'LineWidth',1.2);
xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
title('Portadora c(t) — Dominio del Tiempo');
xlim([0 0.5]); grid on;

subplot(2,2,4);
plot(f_axis/1e3, C_f, 'r', 'LineWidth',1.4);
xlabel('Frecuencia [kHz]'); ylabel('|C(f)|');
title('Espectro de la Portadora — Dominio de la Frecuencia');
xlim([-30 30]); grid on;

sgtitle('FIGURA 1 — Señales Base: Mensaje y Portadora', 'FontSize',13,'FontWeight','bold');

% =========================================================================
%  FIGURA 2: Señal AM modulada (tiempo y frecuencia)
% =========================================================================
figure('Name','Señal AM Modulada','NumberTitle','off','Color','w','Position',[50 50 1200 500]);

subplot(2,1,1);
plot(t*1e3, s_t, 'k', 'LineWidth',1.0); hold on;
plot(t*1e3,  Ac*(1 + mu*m_n), 'r--', 'LineWidth',1.4);
plot(t*1e3, -Ac*(1 + mu*m_n), 'r--', 'LineWidth',1.4);
xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
title(sprintf('Señal AM Modulada s(t)  —  \\mu = %.2f', mu));
legend('s(t) AM','Envolvente superior','Envolvente inferior','Location','northeast');
xlim([0 5]); grid on;

subplot(2,1,2);
plot(f_axis/1e3, S_f, 'm', 'LineWidth',1.4);
xlabel('Frecuencia [kHz]'); ylabel('|S(f)|');
title('Espectro de la Señal AM Modulada');
xlim([-30 30]); grid on;
% Anotación de bandas laterales
text( (fc+fm)/1e3+0.3, max(S_f)*0.85, 'USB','Color','m','FontWeight','bold');
text(-(fc+fm)/1e3-2.0, max(S_f)*0.85, 'LSB','Color','m','FontWeight','bold');
text( fc/1e3+0.3,      max(S_f)*0.95, 'Portadora','Color','r');

sgtitle('FIGURA 2 — Señal AM: Tiempo y Frecuencia', 'FontSize',13,'FontWeight','bold');

% =========================================================================
%  SECCIÓN 5: ANÁLISIS DE RUIDO (AWGN)
%  Se prueba con tres niveles de SNR: 20 dB, 10 dB, 3 dB
% =========================================================================

SNR_vals  = [20, 10, 3];          % SNR objetivo [dB]
colores   = {'#2196F3','#FF9800','#F44336'};

figure('Name','Impacto del Ruido','NumberTitle','off','Color','w','Position',[100 50 1300 700]);
sgtitle('FIGURA 3 — Impacto del Ruido AWGN sobre la Señal AM', 'FontSize',13,'FontWeight','bold');

for k = 1:length(SNR_vals)
    snr_db  = SNR_vals(k);
    s_noisy = awgn(s_t, snr_db, 'measured');  % Agrega ruido AWGN

    % FFT de la señal ruidosa
    Sn_f = fftshift(abs(fft(s_noisy, N) / N));

    % SNR medido
    potencia_onda senoidal = mean(s_t.^2);
    potencia_ruido = mean((s_noisy - s_t).^2);
    snr_medido     = 10*log10(potencia_ondasenoidal / potencia_ruido);

    fprintf('[RUIDO] SNR objetivo: %2d dB  |  SNR medido: %.2f dB\n', snr_db, snr_medido);

    % Gráfica tiempo
    subplot(3,2,2*k-1);
    plot(t*1e3, s_noisy, 'Color', colores{k}, 'LineWidth', 0.6); hold on;
    plot(t*1e3, s_t,     'k--', 'LineWidth', 0.9);
    xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
    title(sprintf('Señal con Ruido — SNR = %d dB (medido: %.1f dB)', snr_db, snr_medido));
    legend('Con ruido','Original','Location','northeast');
    xlim([0 3]); grid on;

    % Gráfica frecuencia
    subplot(3,2,2*k);
    plot(f_axis/1e3, Sn_f, 'Color', colores{k}, 'LineWidth', 1.0); hold on;
    plot(f_axis/1e3, S_f, 'k--', 'LineWidth', 0.8);
    xlabel('Frecuencia [kHz]'); ylabel('|S(f)|');
    title(sprintf('Espectro con Ruido — SNR = %d dB', snr_db));
    legend('Con ruido','Original','Location','northeast');
    xlim([-30 30]); grid on;
end

% =========================================================================
%  SECCIÓN 6: ANÁLISIS DE DISTORSIÓN NO LINEAL
%  Se simula compresión tipo amplificador saturado
% =========================================================================

fprintf('\n[DISTORSIÓN] Análisis de distorsión no lineal...\n');

% Modelo de distorsión: y = x + a2*x^2 + a3*x^3
a2 = 0.1;   % Coeficiente de 2do orden
a3 = 0.05;  % Coeficiente de 3er orden (compresión)

s_dist = s_t + a2*s_t.^2 + a3*s_t.^3;   % Señal distorsionada

% Cálculo de Distorsión Armónica Total (THD simplificado)
S_dist_f = abs(fft(s_dist, N));
f_pos     = (0:N/2-1)*(fs/N);

% Potencia en fundamental y armónicos (±20 Hz alrededor de cada pico)
buscar_pico = @(f0) sum(S_dist_f(round(f0/fs*N)-10 : round(f0/fs*N)+10).^2);
P1 = buscar_pico(fc);
P2 = buscar_pico(2*fc);
P3 = buscar_pico(3*fc);
THD = 100 * sqrt(P2 + P3) / sqrt(P1);
fprintf('[DISTORSIÓN] THD estimado: %.2f %%\n', THD);

figure('Name','Distorsión','NumberTitle','off','Color','w','Position',[150 50 1200 500]);

subplot(2,1,1);
plot(t*1e3, s_dist, 'Color','#9C27B0', 'LineWidth',0.8); hold on;
plot(t*1e3, s_t,    'k--', 'LineWidth',1.0);
xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
title(sprintf('Distorsión No Lineal — a2=%.2f, a3=%.2f  |  THD≈%.2f%%', a2, a3, THD));
legend('Señal distorsionada','Señal original','Location','northeast');
xlim([0 3]); grid on;

subplot(2,1,2);
Sd_f = fftshift(abs(fft(s_dist, N)/N));
plot(f_axis/1e3, Sd_f, 'Color','#9C27B0', 'LineWidth',1.2); hold on;
plot(f_axis/1e3, S_f,  'k--', 'LineWidth',0.8);
xlabel('Frecuencia [kHz]'); ylabel('|S(f)|');
title('Espectro con Distorsión — Aparición de Armónicos Adicionales');
legend('Distorsionada','Original');
xlim([-70 70]); grid on;
% Marcar armónicos
for harm = [1 2 3]
    xline( harm*fc/1e3, 'r:', sprintf('%d°', harm), 'LabelVerticalAlignment','bottom');
    xline(-harm*fc/1e3, 'r:');
end

sgtitle('FIGURA 4 — Distorsión No Lineal', 'FontSize',13,'FontWeight','bold');

% =========================================================================
%  SECCIÓN 7: ANÁLISIS DE ATENUACIÓN
%  Se simula pérdida de canal con distintos factores de atenuación
% =========================================================================

fprintf('\n[ATENUACIÓN] Análisis de atenuación progresiva...\n');

atenuaciones = [1.0, 0.5, 0.2, 0.05];  % Factores de atenuación

figure('Name','Atenuación','NumberTitle','off','Color','w','Position',[200 50 1200 600]);
sgtitle('FIGURA 5 — Efecto de Atenuación de Canal', 'FontSize',13,'FontWeight','bold');

colores_at = {'#1B5E20','#388E3C','#F57F17','#B71C1C'};

for k = 1:length(atenuaciones)
    alpha   = atenuaciones(k);
    s_aten  = alpha * s_t;

    potencia_aten_dB = 20*log10(alpha);
    fprintf('  Atenuación α=%.2f → Pérdida: %.1f dB\n', alpha, abs(potencia_aten_dB));

    subplot(2,2,k);
    plot(t*1e3, s_aten, 'Color', colores_at{k}, 'LineWidth', 0.9); hold on;
    plot(t*1e3, s_t,    'k--', 'LineWidth', 0.7, 'Alpha', 0.5);
    xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
    title(sprintf('α = %.2f  (%.1f dB de pérdida)', alpha, abs(potencia_aten_dB)));
    legend('Atenuada','Original','Location','northeast');
    xlim([0 3]);
    ylim([-Ac*1.5 Ac*1.5]);
    grid on;
end

% =========================================================================
%  SECCIÓN 8: COMPARACIÓN DE ÍNDICES DE MODULACIÓN
%  Sobremodulación (mu > 1) y sus efectos
% =========================================================================

fprintf('\n[ÍNDICE] Comparación de índices de modulación...\n');

mu_vals  = [0.3, 0.7, 1.0, 1.5];
etiquetas = {'Sub-modulación (0.3)','Modulación media (0.7)',...
             'Modulación total (1.0)','Sobremodulación (1.5)'};
colores_mu = {'#01579B','#00695C','#E65100','#B71C1C'};

figure('Name','Índices de Modulación','NumberTitle','off','Color','w','Position',[250 50 1300 600]);
sgtitle('FIGURA 6 — Efecto del Índice de Modulación μ', 'FontSize',13,'FontWeight','bold');

for k = 1:length(mu_vals)
    mu_k  = mu_vals(k);
    s_k   = Ac * (1 + mu_k * m_n) .* cos(2*pi*fc*t);

    subplot(2,2,k);
    plot(t*1e3, s_k, 'Color', colores_mu{k}, 'LineWidth', 0.8); hold on;
    env_pos =  Ac*(1 + mu_k*m_n);
    env_neg = -Ac*(1 + mu_k*m_n);
    plot(t*1e3, env_pos, 'k--', 'LineWidth',1.0);
    plot(t*1e3, env_neg, 'k--', 'LineWidth',1.0);
    xlabel('Tiempo [ms]'); ylabel('Amplitud [V]');
    title(sprintf('μ = %.1f — %s', mu_k, etiquetas{k}));
    xlim([0 4]); grid on;

    % Eficiencia de potencia AM: eta = mu^2 / (2 + mu^2)
    eta = (mu_k^2 / (2 + mu_k^2)) * 100;
    fprintf('  μ = %.1f | Eficiencia de potencia: %.1f%%\n', mu_k, eta);
end

% =========================================================================
%  SECCIÓN 9: TABLA RESUMEN DE MÉTRICAS
% =========================================================================

fprintf('\n==================================================\n');
fprintf('  RESUMEN DE MÉTRICAS\n');
fprintf('==================================================\n');
fprintf('  Parámetro                        Valor\n');
fprintf('  %-32s %s\n', '--------------------------------', '----------');
fprintf('  %-32s %.4f V\n', 'Potencia señal AM (RMS)',  sqrt(mean(s_t.^2)));
fprintf('  %-32s %.4f W\n', 'Potencia media AM',        mean(s_t.^2));
fprintf('  %-32s %.2f %%\n','Eficiencia AM (μ=%.2f)',   (mu^2/(2+mu^2))*100);
fprintf('  %-32s %.2f %%\n','THD estimado (distorsión)',THD);
fprintf('  %-32s %d Hz\n',  'Ancho de banda AM (2*fm)', 2*fm);
fprintf('==================================================\n\n');
fprintf('[OK] Script ejecutado exitosamente. Revisa las 6 figuras generadas.\n');

% =========================================================================
%  FIN DEL SCRIPT
% =========================================================================
