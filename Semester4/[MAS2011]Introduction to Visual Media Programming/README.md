# robot arm 20201127 이창현


    import pygame
    import numpy as np
    
    RED = (255, 0, 0)
    
    FPS = 60   # frames per second
    
    WINDOW_WIDTH = 1400
    WINDOW_HEIGHT = 800
    
    def CirclesOverlap (c1, c2):
        dist12 = np.sqrt( (c1.position[0] - c2.position[0])**2 + (c1.position[1] - c2.position[1])**2 )
        if dist12 < c1.radius + c2.radius:
            return True # if overlaps
        return False
    
    class MyCircle():
        def __init__(self, x, y, vx, vy, radius=40, color=None, sound = None):
            self.radius = radius 
            self.x = x
            self.y = y
            self.vx = vx
            self.vy = vy 
            self.ax = 0
            self.ay = .1
            self.org_color = color if color is not None else (100, 200, 255)
            self.color = self.org_color
            self.sound = sound
            return
        
        def update(self):
            
            self.x = self.x + self.vx 
            if self.x + self.radius >= WINDOW_WIDTH:
                self.vx = -1. * self.vx 
            if self.x - self.radius < 0:
                self.vx = -1. * self.vx 
    
            self.y = self.y + self.vy
            self.vy = self.vy + self.ay 
            if self.y < 0:
                self.vy = 0
                self.y = self.radius + 10
            if self.y + self.radius >= WINDOW_HEIGHT:
                self.vy = -1 * self.vy 
                if self.sound is not None:
                    # self.sound.play()
                    pass
            return
        
        def draw(self, screen):
            pygame.draw.circle(screen, self.color, 
                               [self.x, self.y], 
                               radius = self.radius, )
    
        def print(self,):
            print("my class self.number = ", self.number)
            return
    # 
    
    def getRegularPolygon(nV, radius=1.):
        angle_step = 360. / nV 
        half_angle = angle_step / 2.
    
        vertices = []
        for k in range(nV):
            degree = angle_step * k 
            radian = np.deg2rad(degree + half_angle)
            x = radius * np.cos(radian)
            y = radius * np.sin(radian)
            vertices.append( [x, y] )
        #
        print("list:", vertices)
    
        vertices = np.array(vertices)
        print('np.arr:', vertices)
        return vertices
    
    
    
    class myTriangle():
        def __init__(self, radius=50, color=(100,0,0), vel=[5.,0]):
            self.radius = radius
            self.vertices = getRegularPolygon(3, radius=self.radius)
    
            self.color = color
    
            self.angle = 0.
            self.angvel = np.random.normal(5., 7)
    
            self.position = np.array([0.,0]) #
            # self.position = self.vertices.sum(axis=0) # 2d array
            self.vel = np.array(vel)
            self.tick = 0
    
        def update(self,):
            self.tick += 1
            self.angle += self.angvel
            self.position += self.vel
    
            if self.position[0] >= WINDOW_WIDTH:
                self.vel[0] = -1. * self.vel[0]
    
            if self.position[0] < 0:
                self.vel[0] *= -1.
    
            if self.position[1] >= WINDOW_HEIGHT:
                self.vel[1] *= -1.
    
            if self.position[1] < 0:
                self.vel[1] *= -1
    
            # print(self.tick, self.position)
    
            return
    
        def draw(self, screen):
            R = Rmat(self.angle)
            points = self.vertices @ R.T + self.position
            pygame.draw.polygon(screen, self.color, points)
    #
    
    class myPolygon():
        def __init__(self, nvertices = 3, radius=70, color=(100,0,0), vel=[5.,0]):
            self.radius = radius
            self.nvertices = nvertices
            self.vertices = getRegularPolygon(self.nvertices, radius=self.radius)
    
            self.color = color
            self.color_org = color 
    
            self.angle = 0.
            self.angvel = np.random.normal(5., 7)
    
            self.position = np.array([0.,0]) #
            # self.position = self.vertices.sum(axis=0) # 2d array
            self.vel = np.array(vel)
            self.tick = 0
    
        def update(self,):
            self.tick += 1
            self.angle += self.angvel
            self.position += self.vel
    
            if self.position[0] >= WINDOW_WIDTH:
                self.vel[0] = -1. * self.vel[0]
    
            if self.position[0] < 0:
                self.vel[0] *= -1.
    
            if self.position[1] >= WINDOW_HEIGHT:
                self.vel[1] *= -1.
    
            if self.position[1] < 0:
                self.vel[1] *= -1
    
            # print(self.tick, self.position)
    
            return
    
        def draw(self, screen):
            R = Rmat(self.angle)
            points = self.vertices @ R.T + self.position
    
            pygame.draw.polygon(screen, self.color, points)
    #
    
    def update_list(alist):
        for a in alist:
            a.update()
    #
    def draw_list(alist, screen):
        for a in alist:
            a.draw(screen)
    #
    
    def Rmat(degree):
        rad = np.deg2rad(degree) 
        c = np.cos(rad)
        s = np.sin(rad)
        R = np.array([ [c, -s, 0],
                       [s,  c, 0], [0,0,1]])
        return R
    
    def Tmat(tx, ty):
        Translation = np.array( [
            [1, 0, tx],
            [0, 1, ty],
            [0, 0, 1]
        ])
        return Translation
    #
    
    def draw(P, H, screen, color=(100, 200, 200)):
        R = H[:2,:2]
        T = H[:2, 2]
        Ptransformed = P @ R.T + T 
        pygame.draw.polygon(screen, color=color, 
                            points=Ptransformed, width=3)
        return
    #
    
    
    def main():
        pygame.init() # initialize the engine
        screen = pygame.display.set_mode( (WINDOW_WIDTH, WINDOW_HEIGHT) )
        pygame.display.set_caption("20201127 이창현")
        clock = pygame.time.Clock()
    
        # arm shape
        w = 150
        h = 40
        X = np.array([ [0,0], [w, 0], [w, h], [0, h] ])
    
        # hand shape
        w_hand = 10
        h_hand = 50
        HAND = np.array([ [0,0], [w_hand, 0], [w_hand, h_hand], [0, h_hand]])
    
        position = [WINDOW_WIDTH/2, WINDOW_HEIGHT - 100]
        
        base_delta_y = 0
        base_delta_x = 0
    
        jointangle1 = 0
        jointangle2 = 0
        jointangle3 = 0 
        jointangle4 = -90
        jointangle5 = -90
    
        jointangle1_delta = 0
        jointangle2_delta = 0
        jointangle3_delta = 0
        jointangle4_delta = 0
        jointangle5_delta = 0
    
    
        tick = 0
        done = False
    
        while not done:
            tick += 1
            #  input handling
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    done = True
    
            # drawing
            screen.fill( (200, 254, 219))
    
            # keyboard input
            keys = pygame.key.get_pressed()
    
            if keys[pygame.K_UP]:
                base_delta_y -= 2
    
            if keys[pygame.K_DOWN]:
                base_delta_y += 2
    
            if keys[pygame.K_LEFT]:
                base_delta_x -= 2
    
            if keys[pygame.K_RIGHT]:
                base_delta_x += 2
            
            if keys[pygame.K_q]:
                jointangle1_delta -= 1
    
            if keys[pygame.K_w]:
                jointangle1_delta += 1
    
            if keys[pygame.K_a]:
                jointangle2_delta -= 1
    
            if keys[pygame.K_s]:
                jointangle2_delta += 1
    
            if keys[pygame.K_z]:
                jointangle3_delta -= 1
    
            if keys[pygame.K_x]:
                jointangle3_delta += 1
    
            if keys[pygame.K_SPACE]:
                jointangle4 = -70
                jointangle5 = -110
            else :
                jointangle4 = -90
                jointangle5 = -90
    
            # base
            H0 = Tmat(position[0], position[1]) @ Tmat(0, -h) @Tmat(base_delta_x,base_delta_y)
            draw(X, H0, screen, (0,0,0)) # base
    
            # arm 1
            H1 = H0 @ Tmat(w/2, 0)  
            x1, y1 = H1[0,2], H1[1,2] # joint position
            H10 = H1 @ Rmat(-90) @ Tmat(0,-h/2)
            H11 = H10 @ Tmat(0, h/2) @ Rmat(jointangle1+jointangle1_delta) @ Tmat(0, -h/2)    
            draw(X, H11, screen, (200,200,0)) # arm 1, 90 degree
    
            # arm 2
            H2 = H11 @ Tmat(w, 0) @ Tmat(0, h/2) # joint 2
            x2, y2 = H2[0,2], H2[1,2]
            H21 = H2 @ Rmat(jointangle2+jointangle2_delta) @ Tmat(0, -h/2)
            draw(X, H21, screen, (0,0, 200))
    
            # arm 3
            H3 = H21 @ Tmat(w, 0) @ Tmat(0, h/2) # joint 3
            x3, y3 = H3[0,2], H2[1,2]
            H31 = H3 @ Rmat(jointangle3+jointangle3_delta) @ Tmat(0, -h/2)
            draw(X, H31, screen, (0,0, 200))
    
            # hand 1
            H4 = H31 @ Tmat(w, 0) @ Tmat(0, h/2) # joint 4
            x4, y4 = H4[0,2], H2[1,2]
            H41 = H4 @ Tmat(0,-h_hand/2)
            draw(HAND, H41, screen, (0,0, 200))
    
            # hand 2
            H5 = H41 @ Tmat(w_hand/2,0) #
            H51 = H5 @ Rmat(jointangle4) @ Tmat(-w_hand/2,0)
            draw(HAND, H51, screen, (0,0, 200))
    
            # hand 3
            H6 = H41 @ Tmat(w_hand/2,0) @ Tmat(0,h_hand)
            H61 = H6 @ Rmat(jointangle5) @ Tmat(-w_hand/2,0)
            draw(HAND, H61, screen, (0,0, 200))
            
            # finish
            pygame.display.flip()
            clock.tick(FPS)
    
        # end of while
    # end of main()
    
    if __name__ == "__main__":
        main()
