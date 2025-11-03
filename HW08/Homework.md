- Настроили вывод блокировок долдбше 200мс в журнал. alter system set log_lock_waits = 'on'; alter system set log_min_duration_statement = '200';
- Создали блокировку - запустили в 1 сессии update, во 2 vacuum full. <img width="1370" height="178" alt="image" src="https://github.com/user-attachments/assets/bd3c3d64-1d59-4edf-a7e2-891eada83395" />
- Записи о блокировках попали в лог <img width="1088" height="147" alt="image" src="https://github.com/user-attachments/assets/56c156a5-7b0d-46a2-81f2-cb6b1f8aa007" />
- Создали взаимоблокировку  <img width="1903" height="275" alt="image" src="https://github.com/user-attachments/assets/2b340d92-8dc4-4d33-a041-2af64377c2ff" />
- Записи о дедлоке и запросах его вызвавших остались в логах <img width="1261" height="226" alt="image" src="https://github.com/user-attachments/assets/467f20a7-0592-48e1-a5e1-42531eddafa6" />

