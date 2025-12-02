# Priority Inversion ve Priority Inheritance

## Priority Inversion Nedir?

Priority inversion, yüksek öncelikli bir görevin (High), düşük öncelikli bir görevin (Low) tuttuğu bir kaynağı beklemesi ve bu sırada orta öncelikli görevin (Medium) çalışmaya devam ederek sistemi kilitlemesidir.

### Örnek Senaryo
- **Low task** bir mutex alıyor ve kaynağı kullanıyor.
- **High task** çalışıyor ve aynı mutexi almak istiyor → Low’un bırakmasını bekliyor.
- Bu sırada **Medium task** sürekli çalışıyor ve Low’un mutexi bırakmasına fırsat vermiyor.
- Sonuç: High task, Medium task yüzünden engellenmiş oluyor → *priority inversion*.

## Priority Inheritance Nedir?

Priority inheritance, priority inversion problemine çözüm olarak geliştirilmiştir.  
Eğer düşük öncelikli bir task, yüksek öncelikli task’ın beklediği bir mutexi tutuyorsa:

- Sistem düşük öncelikli task’ın önceliğini *geçici olarak yüksek task seviyesine yükseltir*.
- Böylece düşük öncelikli task işi daha hızlı bitirip mutexi bırakır.
- Ardından önceliği normale döner.

### Örnek Senaryo (Çözüm)
- **Low task** mutexi aldı.
- **High task** mutexi talep etti → bekliyor.
- Sistem: Low task'ın önceliğini High seviyesine yükseltir.
- Low task hızlıca çalışır → mutexi bırakır.
- High task mutexi alır ve çalışmaya devam eder.
- Low task’ın önceliği eski haline döner.

## Sonuç
- **Priority inversion**: Yüksek öncelikli task düşük öncelikli task yüzünden dolaylı şekilde engellenir.
- **Priority inheritance**: Düşük öncelikli task geçici olarak yükseltilerek bu engel ortadan kaldırılır.

