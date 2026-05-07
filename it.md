                # SỬA HÀM THÊM MÓN
                def admin_add_product(name, category, price, image_filename):
                    conn = get_connection()
                    cursor = conn.cursor()
                    cursor.execute('''
                        INSERT INTO product (name, category, price, status, image)
                        VALUES (?, ?, ?, 1, ?)
                    ''', (name, category, price, image_filename))
                    conn.commit()
                    conn.close()
                    return True
                
                # SỬA HÀM CẬP NHẬT MÓN
                def admin_update_product(pid, name, category, price, image_filename):
                    conn = get_connection()
                    cursor = conn.cursor()
                    
                    if not image_filename or image_filename == 'default_food.png':
                        cursor.execute('''
                            UPDATE product SET name = ?, category = ?, price = ? WHERE id = ?
                        ''', (name, category, price, pid))
                    else:
                        cursor.execute('''
                            UPDATE product SET name = ?, category = ?, price = ?, image = ? WHERE id = ?
                        ''', (name, category, price, image_filename, pid))
                        
                    conn.commit()
                    conn.close()
                    return True
